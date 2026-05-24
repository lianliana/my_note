# Motiff MCP Server + UI Generation Skill 协作机制

> 文档路径：tutor-motiff-mcp-server + tutor-electron-student/.claude/skills/motiff-ui-generation

---

## 一、整体架构

```
Motiff 设计平台
      ↓  (Puppeteer 浏览器自动化)
tutor-motiff-mcp-server
      ↓  (MCP 协议 / FastMCP)
Claude Code
      ↓  (Skill 工作流驱动)
motiff-ui-generation skill
      ↓
生成的 React/Vue/Angular 组件代码
```

两者的分工非常清晰：
- **MCP Server**：数据层，负责从 Motiff 平台提取节点数据（JSON 结构 + 截图）
- **Skill**：工作流层，负责驱动 Claude 按固定流程将设计数据转换为代码

---

## 二、MCP Server（tutor-motiff-mcp-server）

### 技术栈
- **FastMCP** ^1.20.5 — MCP 框架
- **Puppeteer** ^24.4.0 — 浏览器自动化，操控 Motiff Web 端
- **TypeScript** + esbuild

### 对外暴露的 MCP Tools

| Tool | 作用 | 关键参数 |
|------|------|----------|
| `getNode` | 获取单个节点的完整 JSON 结构（含子树） | projectId, nodeId, maxDepth, excludeTypes |
| `getNodes` | 批量获取多个节点 JSON | projectId, nodeIds[], saveDir |
| `getPageNodes` | 获取页面浅层节点树（最多 3 层），用于浏览页面结构 | projectId, nodeId |

### 浏览器内注入函数详解

MCP Server 的核心技巧是**把 TypeScript 函数序列化后注入进浏览器页面**，让它们能直接访问 Motiff 平台在 `window` 上挂载的私有 API。

#### window.motiff —— Motiff 官方暴露的只读 API

这是 Motiff 网页应用自己挂载的全局对象，MCP Server 不创建它，只等它出现：

```typescript
// PuppeteerService.ts — waitForMotiffObject()
await this.page.waitForFunction(
  "window.motiff && window.motiff.currentPage && window.motiff.getNodeById"
);
```

它暴露的关键属性：

| 属性/方法 | 说明 |
|-----------|------|
| `window.motiff.currentPage` | 当前设计页面对象，包含完整节点树 |
| `window.motiff.currentPage.children` | 页面顶层节点数组 |
| `window.motiff.currentPage.selection` | 当前选中节点数组 |
| `window.motiff.getNodeById(id)` | 按 ID 直接取节点（不一定存在） |

**重要**：`window.motiff` 上的节点对象属性挂在原型链上而非自身属性，直接 `JSON.stringify` 会丢失数据，这也是需要自定义提取函数的根本原因。

---

#### 函数注入机制 —— mountFunctions()

`PuppeteerService.mountFunctions()` 把 `motiffExtractor.ts` 导出的函数对象逐个序列化注入页面：

```typescript
// PuppeteerService.ts
await this.page?.evaluate(
  (functionData: { key: string; func: string }) => {
    window[functionData.key] = new Function(`return ${functionData.func}`)();
  },
  { key, func: func.toString() }   // func.toString() 把函数体变成字符串传过去
);
```

注入时机有两处：
1. **服务初始化时**：`MotiffService.initialize()` → `mountFunctions()`
2. **页面导航后**：`waitForMotiffLoaded()` 成功后再次调用 `mountFunctions()`（防止页面刷新把注入的函数清掉）

注入后页面全局多出 4 个函数：

```
window.getObjectPropertiesAsJson
window.extractNodeInfo
window.findNodeById
window.getPageData
```

---

#### window.getObjectPropertiesAsJson —— 原型链属性提取器

**为什么需要它**：Motiff 节点对象的属性（`id`、`width`、`fills` 等）挂在原型链上，`Object.keys()` 取不到，必须遍历原型链才能拿全。

```
节点对象本身 (own properties)
    ↓ Object.getPrototypeOf()
原型1（包含 id, type, name…）
    ↓ Object.getPrototypeOf()
原型2（包含 width, height, x, y…）
    ↓ Object.getPrototypeOf()
...（最多遍历 3 层，maxDepth 默认值）
```

核心逻辑：

```typescript
// 遍历原型链收集所有属性名
while (currentObj && depth < maxDepth) {
  Object.getOwnPropertyNames(currentObj).forEach(prop => {
    if (!skipProps.includes(prop)) allProps.add(prop);
  });
  currentObj = Object.getPrototypeOf(currentObj);
  depth++;
}

// 再从原始对象 obj 直接读值（不是从原型读）
allProps.forEach(prop => {
  const value = obj[prop];   // JS 原型链查找，自动找到正确层级
  // ...过滤逻辑
});
```

**白名单/黑名单**：
- `importantProps`：保留对 AI 生成 UI 有用的属性（`id/type/name/children/fills/fontSize/layoutMode/padding*` 等）
- `skipProps`：过滤掉对 AI 无意义的噪声（`parent/relativeTransform/blendMode/isMask` 等）

---

#### window.extractNodeInfo —— 递归节点提取器

以 `getObjectPropertiesAsJson` 为基础，递归处理整棵子树：

```typescript
function extractNodeInfo(node, depth = 0, maxDepth = 50) {
  const nodeInfo = getObjectPropertiesAsJson(node);  // 提取当前节点属性

  if (nodeInfo.children && Array.isArray(nodeInfo.children)) {
    nodeInfo.children = nodeInfo.children.map(child =>
      extractNodeInfo(child, depth + 1, maxDepth)    // 递归处理每个子节点
    );
  }

  return nodeInfo;
}
```

`maxDepth` 默认 50，对应 MCP Tool 的 `maxDepth` 参数，防止深度嵌套的设计文件撑爆 context。

---

#### window.findNodeById —— 节点 ID 搜索

在节点树中按 ID 查找，优先走三条快捷路径：

```typescript
function findNodeById(root, nodeId) {
  // 快捷路径1：当前选中节点（最快）
  const currentSelection = window.motiff.currentPage.selection[0];
  if (currentSelection?.id === nodeId) return currentSelection;

  // 快捷路径2：根节点本身
  if (root.id === nodeId) return root;

  // 快捷路径3：递归 children
  for (const child of root.children ?? []) {
    if (child.id === nodeId || child.id.includes(nodeId)) return child;
    const found = findNodeById(child, nodeId);
    if (found) return found;
  }

  return null;
}
```

实际上 `MotiffService.getNodeById()` 优先用 `window.motiff.getNodeById(id)`（Motiff 官方 API，O(1) 查找），只有官方 API 不存在时才退化到 `findNodeById` 递归搜索。

---

#### window.getPageData —— 页面完整数据序列化

专门用于把 `window.motiff.currentPage` 整体序列化为纯 JSON（解决原型链 + 循环引用问题）：

```typescript
function getPageData() {
  function getProps(obj) {
    const result = {};
    for (const key in obj) {   // for...in 可以枚举原型链属性
      const value = obj[key];
      if (typeof value !== "function" && value !== undefined) {
        if (Array.isArray(value)) {
          result[key] = value.map(item =>
            typeof item === "object" ? getProps(item) : item
          );
        } else if (typeof value === "object" && value !== null) {
          result[key] = getProps(value);
        } else {
          result[key] = value;
        }
      }
    }
    return result;
  }
  return getProps(window.motiff.currentPage);
}
```

与 `getObjectPropertiesAsJson` 的区别：
- `getObjectPropertiesAsJson`：有白名单/黑名单过滤，适合单节点精准提取
- `getPageData`：全量序列化，不过滤，适合 `getAllNodes()` 这类需要完整树结构的场景

---

### 四个函数的调用关系图

```
MotiffService.getNodeById(nodeId)
  └─ page.evaluate(() => {
       // 1. 先尝试官方 API
       window.motiff.getNodeById(id)          ← window.motiff（官方）
       // 2. 兜底用选中节点
       window.motiff.currentPage.selection    ← window.motiff（官方）
       // 3. 拿到节点后提取属性
       window.extractNodeInfo(node)           ← 注入函数
         └─ window.getObjectPropertiesAsJson(node)  ← 注入函数（遍历原型链）
            └─ 递归处理 children
     })

MotiffService.getPageNodes(maxDepth)
  └─ page.evaluate(() => {
       window.motiff.currentPage.children     ← window.motiff（官方，直接读）
       // 浅层遍历，不调用注入函数，直接访问 node.id/type/name 等
     })

MotiffService.getCurrentPageData() / getAllNodes()
  └─ page.evaluate(() => {
       window.getPageData()                   ← 注入函数
         └─ window.motiff.currentPage         ← window.motiff（官方）
            └─ getProps() 递归全量序列化
     })
```

---

### 核心数据处理流程

```
getNode(projectId, nodeId)
  → 导航到 https://beta.motiff.cn/file/{projectId}?type=dev&nodeId={nodeId}
  → waitForFunction("window.motiff && window.motiff.currentPage && window.motiff.getNodeById")
  → mountFunctions() 注入 4 个辅助函数到页面 window
  → page.evaluate():
      window.motiff.getNodeById(id)         // 官方 API 取原始节点对象
        └─ window.extractNodeInfo(node)     // 遍历原型链提取有效属性
             └─ 递归处理 children
      JSON.parse(JSON.stringify(nodeInfo))  // 深拷贝脱离原型链，变成纯对象
  → processNode()（Node.js 侧）：
      - 过滤无用类型（LINE / VECTOR / BOOLEAN_OPERATION）
      - RGB 转 hex
      - padding/borderRadius 合并为 CSS shorthand
      - flex 布局标准化
      - 递归处理 children
  → compressNodeData()（Node.js 侧）：
      - 对重复 instance 去重（只保留 referenceInstanceId 指针）
      - key 名压缩（id→i, children→c, type→t…）
      - type 名压缩（FRAME→F, TEXT→T…）
      - 附带 metadata 描述压缩方案
  → 返回压缩后的 JSON 给 Claude
```

压缩策略类似 Lottie JSON，可降低约 50%+ 体积，减少 Claude context 消耗。

### 配置方式（.env）

```env
TRANSPORT_TYPE=stdio        # 与 Claude Code CLI 配合用 stdio
MOTIFF_URL=https://beta.motiff.cn
MOTIFF_USERNAME=xxx@xxx.com
MOTIFF_PASSWORD=xxxxxxxx
HEADLESS=true
```

---

## 三、UI Generation Skill（motiff-ui-generation）

Skill 是 Claude Code 的工作流脚本，定义了 Claude 必须严格遵循的 5 阶段流程。

### 5 阶段流程总览

```
Phase 1: 截图分析 → 生成 planning-doc.md
    ↓
Phase 2: 数据预取 → 生成 node-mapping.md + .motiff/ 目录
    ↓  ← 强制暂停，等待用户确认
Phase 3: 完整页面生成 → index.tsx + 样式文件 + example.tsx
    ↓
Phase 4: UI 迭代循环（按需多次）
    ↓  ← 强制暂停，等待用户满意
Phase 5: 组件拆分（用户主动触发）
```

### 各阶段详解

#### Phase 1 — 截图分析
- 调用 `mcp_motiff-official_get_motiff_node_screenshot` 获取根节点截图
- Claude 视觉分析：布局结构、组件边界、重复元素、状态变体
- 产物：`planning-doc.md`（Props 接口设计、子组件划分方案）
- **截图是唯一布局权威**，JSON/HTML 数据仅辅助参考

#### Phase 2 — 数据预取
- 执行 CLI：`npx tsx motiff-prefetch/cli.ts prefetch`（批量拉截图 + HTML）
- 调用 `mcp_motiff_getNodes`（MCP Server 的 getNodes tool）批量拉 JSON
- 创建 `.motiff/` 目录，按节点 ID 组织数据
- 产物：`node-mapping.md`（NodeId ↔ 页面区域的映射表）
- **强制暂停**：Phase 3 前必须等用户确认

#### Phase 3 — 完整页面生成
- 严格只读根节点数据（不读子组件数据）
- 产物：`index.tsx`（完整页面）、样式文件、`example.tsx`（mock 数据预览）
- 原则："先完整，后拆分"

#### Phase 4 — UI 迭代
- 用户指出问题区域 → 查 `node-mapping.md` 定位对应子组件
- 只读该子组件的截图/HTML/JSON，精准修改
- 循环直到用户满意

#### Phase 5 — 组件拆分（可选）
- 提取网络资源到本地 `assets/`
- `shared/`：公共可复用组件
- `components/`：页面区域组件
- 更新 `index.tsx` 为组合式结构
- 产物：`generation-summary.md`（组件接口文档）

### Skill 调用的 MCP Tools 汇总

| Skill 内调用 | 对应 MCP Tool / CLI | 所属 |
|-------------|---------------------|------|
| `mcp_motiff-official_get_motiff_node_screenshot` | Motiff 官方 MCP | 官方插件 |
| `mcp_motiff_getNodes` | `getNodes` | tutor-motiff-mcp-server |
| `npx tsx motiff-prefetch/cli.ts prefetch` | 本地脚本 | 项目内 CLI |
| `npx tsx asset-extractor/cli.ts` | 本地脚本 | 项目内 CLI |

---

## 四、两者协作的核心逻辑

### 数据流向

```
Phase 2 (Skill) 
  → 调用 mcp_motiff_getNodes (MCP Server)
      → Puppeteer 浏览器自动化打开 Motiff
      → 提取节点 JSON → 压缩 → 返回
  → 结果存入 .motiff/{nodeId}/data.json

Phase 3/4 (Skill)
  → Claude 读取 .motiff/ 中的预取数据（不再重复调用 MCP）
  → 对照截图生成/修改代码
```

### 关键设计决策

1. **预取 vs 按需拉取**：Skill 在 Phase 2 一次性拉完所有数据，Phase 3/4 只读本地缓存，避免反复触发浏览器自动化（性能 + 稳定性）

2. **截图 + JSON 分层设计**：截图和 JSON 各自承担不同职责，形成分层结构——截图负责空间布局、组件边界、视觉层次等"用眼睛看就能判断"的内容；JSON 负责颜色、字号、间距、圆角等截图里看不出来的精确数值。这类似于 Skill 本身的设计模式：description 建立宏观意图，具体参数提供细节。Skill 明确规定"截图是唯一布局权威"，原因在于设计稿的视觉层次和节点树的嵌套深度经常对不上，如果让 AI 从 JSON 嵌套结构去推断布局，是最容易出错的地方

3. **数据隔离原则**：Phase 3 只看根节点，Phase 4 只看被修改的子节点，防止 Claude 被无关数据干扰

4. **压缩优化**：MCP Server 的 `compressNodeData` 把节点 JSON 体积压缩约 50%+，在 Claude context window 有限的情况下能处理更大的设计文件。压缩方式是把长字段名替换成短字符（`width→w`、`children→c`、`FRAME→F`），同时在数据顶部带一个 `meta` 映射表：

   ```json
   {
     "meta": {
       "keys": { "w": "width", "c": "children", "t": "type", ... },
       "types": { "F": "FRAME", "T": "TEXT", "I": "INSTANCE", ... }
     },
     "root": { "t": "F", "w": 375, "c": [...] }
   }
   ```

   这里存在一个真实的权衡：理论上模型看 `meta` 能解码全部字段，语义完整；但 LLM 对 token 的理解不是"先查字典再推理"，`width: 375` 这个 token 序列本身就携带训练数据中的语义，而 `w: 375` 需要模型额外做一次映射，会增加理解负担。**不压缩时语义更清晰，压缩时能处理更大文件**——对于复杂页面展开可能有几万 token 的设计文件，不压缩直接超出 context window，这是工程上不得不做的妥协。

5. **强制暂停点**：Phase 2→3、Phase 4→5 均有强制人工确认，防止 AI 在数据不完整或方向错误时继续生成大量无效代码

### 支持的技术栈适配器

| Adapter | 样式方案 |
|---------|----------|
| react-module-scss | React + CSS Modules + SCSS |
| react-unocss | React + Unocss 原子化 CSS |
| vue-module-scss | Vue3 + CSS Modules + SCSS |
| angular-scss | Angular + SCSS |

---

## 五、使用方式

### 前置配置

1. 在 Claude Code 的 MCP 配置中注册 `tutor-motiff-mcp-server`
2. 配置 `.env` 填写 Motiff 账号信息
3. 项目内存在 `motiff-prefetch/cli.ts` 和 `asset-extractor/cli.ts` 脚本

### 触发 Skill

在 Claude Code 中执行：
```
/motiff-ui-generation
```

然后按提示提供：
- **Project ID**：Motiff 项目 ID
- **Node ID**：根页面节点 ID（格式 `123:456`）
- **Target Path**：代码生成目标目录
- **Tech Stack**：选择上述 4 种适配器之一

---

## 六、常见问题

**Q: getPageNodes 和 getNodes 什么场景用哪个？**  
A: `getPageNodes` 最深 3 层，用于快速浏览页面节点树、确认节点 ID；`getNodes` 用于批量深度提取，Skill Phase 2 使用的是后者。

**Q: 为什么 Phase 3 不直接调 MCP 而要读本地缓存？**  
A: 减少 Puppeteer 触发次数（浏览器操作慢），同时保证 Phase 3/4 数据一致性（Phase 2 快照是固定的）。

**Q: 节点 ID 如何找？**  
A: 先用 `getPageNodes` 获取页面浅层结构，在返回树中找到对应区域的节点 ID；或在 Motiff Dev Mode 界面直接复制。
