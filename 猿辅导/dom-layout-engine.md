# 基于 Puppeteer + Range API 的 DOM 布局提取引擎

> 将 ProseMirror 富文本渲染结果序列化为 DomLayout 绝对坐标数据，C 端按坐标还原渲染，解决老旧 Android WebView 排版差异、自定义字体加载时序导致的布局跳变，以及播放器因 contenteditable 带来的移动端交互副作用。

---

## 一、为什么要这么做

### 最简单的方案：直接传 HTML

最直觉的做法是 B 端把 ProseMirror 渲染出的 HTML 字符串存下来，C 端直接丢进浏览器渲染。

这个方案有三个问题：

**1. 老旧 Android WebView 排版差异**

Android 4.x、5.x 的系统 WebView 版本很老，和现代 Chrome 的行高计算、字符间距规则不同。同一段 HTML，老设备渲染出来的换行点可能不一样，图文会错位。课辅产品用户里低端机比例不低，这是真实痛点。

**2. 自定义字体加载时序导致布局跳变**

课件通常用定制字体。C 端字体未加载完时，浏览器先用 fallback 字体占位排版。fallback 字体字宽不同，换行点就变了。字体加载完后触发重排，画面跳变，最终位置也可能和 B 端不同。

**3. contenteditable 的移动端副作用**

ProseMirror 依赖 `contenteditable`。C 端是只读播放器，但带着这个属性，用户长按会弹出系统文字选择菜单，有时还会触发软键盘。这些全是 bug，要花额外代码禁掉。

### 这套方案的核心思路

把"排版计算"这件事从 C 端剥离，让它只发生一次、发生在受控环境（Puppeteer 跑的 headless Chrome）里。

B 端在字体、公式图片全部加载完毕后，用 Range API 测量每行文字的位置和换行点，将结果固化为 DomLayout 绝对坐标数据。

C 端拿到的不是"待排版的内容"，而是"已经排好版的坐标"。C 端只做一件事：**在指定坐标画指定内容**，不做任何排版决策，也不依赖 ProseMirror。

---

## 二、整体架构

```
┌──────────────────────────────────────────────────────────────────────┐
│                     B 端：tutor-cyber-slide-editor                    │
│                                                                        │
│  EditorBox[]  →  ProseMirror 渲染  →  真实 DOM                        │
│                  （字体、公式图片全部加载完毕）                          │
│                                          ↓                             │
│                          createDOMLayoutFromElement()                  │
│                          （Range API 逐行切分 + 字符坐标测量）           │
│                                          ↓                             │
│                     DomLayout 树（绝对坐标、浮点像素值）                 │
│                                          ↓                             │
│                     window.createDOMLayout() 挂载到全局               │
└──────────────────────────────────────────────────────────────────────┘
                                   ↓
             Puppeteer（后端服务）打开截图页，等待 window.allDone = 1
             调用 window.createDOMLayout()，拿到 SlideConfig JSON 写入 CDN
                                   ↓
┌──────────────────────────────────────────────────────────────────────┐
│                     C 端：tutor-box-slide-player                      │
│                                                                        │
│  SlideConfig { layouts: DomLayout[], animations, fontMap }            │
│           ↓                                                            │
│  @tutor/box-slide-renderer                                            │
│  ├─ 遍历 DomLayout 树                                                  │
│  ├─ 每个节点 → position:absolute + 绝对坐标                            │
│  └─ DomText 用 whiteSpace:pre 禁止重新排版                             │
│           ↓                                                            │
│  CSS transform: scale(s)  transformOrigin: 0 0                        │
│  → 适配各机型视口，布局结构保持一致                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 三、B 端：Range API 布局提取引擎

### 3.1 为什么需要 Range API

字体文件里每个字符的宽度由字模坐标决定，经过 `font-size` 缩放后才得到实际宽度。没有任何公式能预测——`font-size: 24px` 下，'i' 可能是 4.7px，'W' 可能是 21.1px。只有浏览器加载字体文件、完成排版后，才能通过 Range API 测量到真实值。

### 3.2 核心函数：`createDOMLayoutFromElement`

**文件：** `projects/cse-core-lib/src/logic/createDomLayout.ts`

递归遍历 DOM 树，将不同节点类型转换为对应的 DomLayout 节点：

```
DOM 节点类型               →  DomLayout 类型
────────────────────────────────────────────
Text node                 →  DomText（按行切分，每行一个）
<img>                     →  DomImage
<img class="tex">         →  DomImage（公式）
<u>                       →  DomLine（下划线）
<uw>                      →  DomWavy（波浪线）
<ud>                      →  DomCircles（着重号圆点）
<p>                       →  DomContainer（段落容器）
.placeholder              →  跳过
```

### 3.3 文本行切分：`getTextSplitLinePos`

同一个 Text 节点的内容可能跨多行显示，需要找到每个换行点的字符偏移量。

**为什么用二分法：**

笨办法是逐字符扫描，15 个字要测 15 次。二分法每次排除一半范围，15 个字只需 4 次，复杂度 O(log n)。

**判断依据：**

```typescript
const range = document.createRange();
range.setStart(node, start);
range.setEnd(node, mid);
const rect = range.getBoundingClientRect();

if (rect.height > lineHeight) {
  // 已跨行，换行点在 [start, mid)
  right = mid;
} else {
  // 未跨行，换行点在 (mid, end]
  left = mid + 1;
}
```

**核心难点：误判问题**

判断依据是 `rect.height` 变大，但高度变大不一定是文字换行——行内公式图片（MathJax 渲染的图片高度大于文字）也会导致行高变化，二分法会在公式图片处误判换行点。这是需要专门处理的 edge case。

另一个陷阱是 Unicode surrogate pair：emoji 和某些生僻字占两个 code unit，如果按字节偏移切而不是按 code point 切，会切到字符中间，Range API 直接报错。

### 3.4 字符坐标提取：`getCharRects`

找到换行点后，逐字符测量 x 偏移，存入 `charX[]`：

```typescript
function getCharRects(node: Text, start: number, end: number): ClientRect[] {
  const rects: ClientRect[] = [];
  for (let i = start; i < end; ) {
    const charEnd = nextCodePoint(node.data, i);  // 按 code point 步进
    const range = document.createRange();
    range.setStart(node, i);
    range.setEnd(node, charEnd);
    rects.push(range.getBoundingClientRect());
    i = charEnd;
  }
  return rects;
}
```

**charX 的用途：**

C 端渲染文字时不逐字符定位（整行文字一次性渲染），charX 的作用是给**着重号（DomCircles）和下划线**定位——需要知道某个具体字符在哪里，才能把圆点精确点在那个字下面。

### 3.5 坐标系与缩放

提取时有两个参数：

| 参数        | 含义                              |
| ----------- | --------------------------------- |
| `viewScale` | 编辑器当前缩放比（用户可放大画布查看） |
| `scale`     | 输出 DomLayout 的目标缩放倍率      |

`getBoundingClientRect()` 返回的屏幕坐标受 `viewScale` 影响，需要修正：

```typescript
const rectScale = scale / viewScale;
// 所有 Range API 坐标 × rectScale → 存入 DomLayout
```

---

## 四、B 端：LayoutGeneratorService

**文件：** `projects/cse-lib/src/services/layout-generator.service.ts`

### 处理管线

```
EditorBox[]（按 zIndex 排序）
    │
    ├─ HtmlBox / ComplexImageBox
    │       └─ querySelector('.ProseMirror')
    │               └─ createDOMLayoutFromElement()  ← Range API 在这里
    │
    ├─ ImageBox  →  generateFromImageBox()
    ├─ SvgBox    →  viewBox 坐标换算 → DomImage(svg)
    ├─ TableBox  →  querySelectorAll('.ProseMirror') × N
    └─ AnimationBox  →  .zip 资源引用 → DomAnimation
    │
    ▼
后处理：
├─ addBoundingRects()         — 旋转元素计算轴对齐包围盒
├─ mergeAdjacentLineAndWavy() — 合并相邻下划线/波浪线
├─ resolveRelativeDOMLayout() — 相对坐标 → 绝对坐标
├─ truncateNumbers(2)         — 坐标保留两位小数（浏览器返回浮点数）
└─ filterSpecialChars()       — 过滤特殊字符
```

---

## 五、DomLayout 数据结构

### 5.1 类型系统

```typescript
type DomLayout =
  | DomContainer   // 容器/分组
  | DomText        // 文本（一行一个节点）
  | DomImage       // 图片（png/jpg/svg/base64）
  | DomLine        // 直线下划线
  | DomWavy        // 波浪线
  | DomCircles     // 着重号（圆点序列）
  | DomAnimation   // Lottie 动画
```

### 5.2 关键字段

**DomText（文本节点，每个节点对应一行）：**

```typescript
{
  type: 'text'
  text: string              // 这一行的完整文字内容
  position: { x, y }       // 行左上角绝对坐标（px，浮点数）
  size: { x, y }            // 宽高（px）
  charX: number[]           // 每个字符的行内 x 偏移，供着重号/下划线定位
  fontSize: number
  fontFamily: string
  fontWeight: string        // 注意：C 端用 text-shadow 模拟加粗，不用 font-weight
  color: string
  lineHeight: number
  letterSpacing: number
}
```

**整体产物：SlideConfig**

```typescript
interface SlideConfig {
  layouts: DomLayout[]              // 每一页 = 一棵 DomLayout 树
  animations: AnimationInfo[]       // 动画序列（步进、持续时间等）
  fontMap: Record<string, string>   // 字体名 → base64 字体数据
  quiz?: IQuiz                      // 习题配置
  scale?: number                    // 提取时的缩放倍率
}
```

---

## 六、Puppeteer 的角色

### 就绪信号机制（核心难点）

Range API 必须在**字体加载完、公式图片渲染完、CSS 完全生效后**才能测量，早一毫秒数据就是错的。

B 端截图页在所有异步资源就绪后才发出信号：

```typescript
window.allDone = 1                              // 通知 Puppeteer 可以测量了
window.createDOMLayout = () => SlideConfig      // 触发 Range API 测量，返回完整数据
window.getRangeRenderThumbnails = () => Buffer  // 返回页面截图
```

### Puppeteer 调用流程

```
后端服务
  │
  ├─ 1. puppeteer.launch()  — 启动 headless Chrome
  ├─ 2. page.goto('/screenshot?slideId=xxx')
  ├─ 3. page.waitForFunction('window.allDone === 1')
  │      等 ProseMirror 排版、公式图片、自定义字体全部 ready
  ├─ 4. page.evaluate('window.createDOMLayout()')
  │      Range API 在此时执行，返回 SlideConfig JSON
  ├─ 5. page.screenshot()  — （可选）捕获缩略图
  └─ 6. 将 SlideConfig 写入 OSS / CDN
```

**为什么必须用 Puppeteer 而不是 Node.js 直接解析：**

Range API 是浏览器 API，必须在真实渲染环境中执行。字体渲染、字距、公式图片尺寸，都依赖浏览器排版引擎，Node.js 里没有这些。

---

## 七、C 端渲染引擎

### 7.1 共享渲染库：@tutor/box-slide-renderer

B 端预览和 C 端播放共用同一个渲染库，避免两套实现产生差异。

核心渲染逻辑：遍历 DomLayout 树，每个节点转换为 `position: absolute` 的 DOM 元素：

```
DomContainer  → <div style="position:absolute; left:x; top:y; width:w; height:h">
DomText       → <div style="position:absolute; left:x; top:y; white-space:pre">
                  整行文字一次性写入 textContent，不逐字符定位
DomImage      → <img style="position:absolute; ...">（支持 clip 裁剪）
DomLine       → <div style="position:absolute; height:lineWidth; background:color">
DomWavy       → 平铺 base64 图片实现波浪线
DomCircles    → N 个 <span style="border-radius:50%; position:absolute">
DomAnimation  → 插件接管（Lottie）
```

### 7.2 文字渲染的关键细节

**为什么不逐字符定位：**

每个 `DomText` 节点本来就只有一行（B 端已经按行切分），直接设 `whiteSpace: pre` 禁止浏览器换行，整行一次渲染即可。字符间距用 `letterSpacing` 统一控制。

**加粗用 text-shadow 模拟：**

```typescript
if (fontWeight === '700') {
  const offset = Math.floor(fontSize / 35) || 1
  container.style.textShadow = `0px ${offset}px 0px ${color}`
  // 再克隆一份右移 offset px 叠加
}
```

原因：不同设备上 `font-weight: bold` 会切换到 bold 字形，字宽和 Regular 不同，会破坏 charX 定位的着重号位置。text-shadow 只改外观，不改字宽。

### 7.3 插件机制

Lottie 动画、Audio 等复杂元素通过插件注册，不硬编码在渲染库里：

```typescript
slideRenderer.registerPlugin('animation', lottiePlugin)
slideRenderer.registerPlugin('audio', audioPlugin)
```

---

## 八、设备适配：CSS transform scale

### 为什么不用 rem/vw

如果把所有坐标乘以缩放比（如 `font-size: 24px → 12px`），字号变了，浏览器会重新排版。C 端设备在 `font-size: 12px` 下算出的字宽和 B 端在 `font-size: 24px` 下量的 charX 不是同一套数，着重号和下划线定位就错了。

`transform: scale` 发生在排版之后的 GPU 合成阶段，字号从未改变，浏览器不重新排版，所有坐标原封不动地保留。

### 实现

课件基准分辨率 1440 × 1080，C 端设备视口宽度 W：

```typescript
const scaleRadio = (container.clientWidth / 1440) * config.scale
element.style.transform = `scale(${scaleRadio})`
element.style.transformOrigin = `0px 0px`   // 从左上角缩放
```

DomLayout 坐标始终是 1440 基准下的像素值，`transform: scale` 整体缩小，相当于把渲染好的画面缩小拍照，所有元素相对位置不变。

### 关于"像素级一致"

这套方案保证的是**布局结构一致**——行不会折行、元素不会位移。文字内部字符的渲染仍然由 C 端设备的字体引擎决定，严格意义上不是像素级完全一致。但相比直接传 HTML，消除了老旧 WebView 和字体加载时序这两个最主要的差异来源。

---

## 九、核心难点总结

**难点一：Range API 测量时机**

必须等字体、公式图片、CSS 全部就绪后才能测量。需要设计可靠的就绪信号机制（`window.allDone`），确保信号在所有异步资源 ready 之后才发出。

**难点二：行切分的 edge case**

二分法判断换行点依赖 `rect.height` 变化，但行内公式图片也会撑高行高，导致误判。加上 Unicode surrogate pair 问题（emoji 等占两个 code unit），需要按 code point 步进而不是按字节切分。

---

## 十、设计决策速查

| 决策 | 原因 |
| ---- | ---- |
| 用 Range API 而非 canvas measureText | 浏览器排版引擎支持 kerning、ligature、CJK，比 measureText 精确 |
| 用 Puppeteer 而非 Node.js 解析 | Range API 必须在真实 DOM 渲染环境执行 |
| 每行切分为独立 DomText | C 端 `whiteSpace:pre` 只需保证一行不折，不需要跨行排版 |
| charX 存字符级 x 偏移 | 供着重号和下划线精确定位，不用于文字本身渲染 |
| 加粗用 text-shadow 模拟 | 避免 bold 字形字宽变化破坏 charX 定位 |
| CSS transform scale 适配设备 | 排版后整体缩放，不触发重排版 |
| BC 端共享渲染库 | 预览和播放用同一套渲染逻辑，避免两套实现产生差异 |
