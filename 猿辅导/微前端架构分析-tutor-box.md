# tutor-box 微前端架构分析

> 涉及项目：`tutor-box-cocos-hall`（主应用基座）、`tutor-box-camera-lib`（React 子应用）、`ccpack`（自研构建后处理工具）

---

## 一、整体架构

这套方案**不使用** webpack Module Federation，而是利用**标准 webpack 的 `webpackJsonp` 共享 runtime 机制**，配合 `@tutor/cocos-compiler` 的构建插件实现模块共享。

Host 先加载并初始化 webpack runtime，子应用通过 `webpackJsonp.push(...)` 将自身模块注册进同一个 runtime，从而天然复用 Host 已加载的所有依赖。

```
┌──────────────────────────────────────────────────────────┐
│  Host: tutor-box-cocos-hall                              │
│                                                          │
│  runtime.js  →  初始化 webpack runtime                   │
│                 installedModules = {}  (全局模块缓存)     │
│                 __webpack_require__ = function(id) {...}  │
│                                                          │
│  vendors~lib.js  →  react / rxjs / mobx / @tutor/... 等  │
│  hall-kernel.js  →  hall 业务模块                        │
│  live-common.js  →  live 业务模块                        │
│  project.js      →  hall 入口逻辑                        │
└──────────────────────────────────────────────────────────┘
                        ↓ 运行时动态加载
┌──────────────────────────────────────────────────────────┐
│  Sub-App: tutor-box-camera-lib                           │
│                                                          │
│  project.js  →  webpackJsonp.push([["main"], {           │
│      "$$app.1003596.154/3": function(e,t) {              │
│          e.exports = arguments[2]("react")               │
│          // arguments[2] = Host 的 __webpack_require__   │
│      }                                                   │
│  }])                                                     │
│  → 直接拿到 Host installedModules 里缓存的 react 实例    │
└──────────────────────────────────────────────────────────┘
```

Host 产物文件：

| 文件 | 内容 | 说明 |
|------|------|------|
| `runtime.js` | webpack bootstrap | 定义 `__webpack_require__`、`installedModules`、`webpackJsonp` |
| `vendors~lib.js` | npm 公共依赖 | react/rxjs/mobx/@tutor/* 等共享包 |
| `hall-kernel.js` | hall 业务模块 | 按 `namedChunkedGroups` 拆出的具名 chunk |
| `live-common.js` | live 业务模块 | 同上 |
| `project.js` | hall 入口逻辑 | Cocos 场景、组件等 |

---

## 二、公共依赖复用

### 2.1 构建阶段 —— NamedModulesPlugin 给模块起包名 ID

Host 使用 `@tutor/cocos-compiler` 内置的 `NamedModulesPlugin`，将 `sharedModule` 配置列表中的包的 webpack 模块 ID 从数字改为包名字符串：

```typescript
// tutor-box-cocos-hall/packages/hall/ccpack.config.ts
sharedModule: [
    '@tutor/vue-application',
    '@tutor/hall-kernel',
    '@tutor/box-cocos-hall-lib',
    '@tutor/box-live-common',
    '@tutor/react-framework',
    'protobufjs/light',
]
```

这样 `installedModules` 里这些模块的 key 就是包名（如 `"react"`、`"@tutor/box-live-common"`），而不是数字，子应用才能通过包名找到它们。

### 2.2 构建阶段 —— 子应用 ExternalsPlugin 改写 import

`tutor-box-camera-lib/config-overrides.js` 声明 `remoteModules`：

```javascript
remoteModules: [
    '@tutor/box-bridge-ts',
    '@tutor/box-live-common',
    'react', 'react-dom', 'mobx', 'rxjs',
    // ...
]
```

`@tutor/ccpack-react-plugin` 注入 webpack `ExternalsPlugin`，将这些包的 import 改写为 `arguments[2](包名)` 的形式：

```javascript
new ExternalsPlugin('root', function(context, request, callback) {
    if (modules.includes(request)) {
        return callback(null, `arguments[2]("${request}")`);
    }
    callback();
})
```

子应用打包结果（dist/src/project.js 实际内容）：

```javascript
// 源码：
import React from 'react'
import { SomeClass } from '@tutor/box-live-common'

// 打包后（ExternalsPlugin 改写）：
"$$app.1003596.154/3": function(e, t) {
    e.exports = arguments[2]("react")
},
"$$app.1003596.154/8": function(e, t) {
    e.exports = arguments[2]("@tutor/box-live-common")
},
```

这些包的代码**完全不打进子应用**，子应用 project.js 体积大幅缩小。

### 2.3 运行时 —— webpackJsonp 共享 runtime

这是整个机制的核心。webpack 调用每个模块工厂函数时，固定传三个参数：

```javascript
// runtime.js 里 __webpack_require__ 的实现
function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
        return installedModules[moduleId].exports;  // 缓存命中，直接返回
    }
    var module = installedModules[moduleId] = { exports: {} }

    // 调用模块工厂，第三个参数传入 __webpack_require__ 自身
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__)
    //                                     ↑arg0   ↑arg1           ↑arg2

    return module.exports;
}
```

子应用通过 `webpackJsonp.push(...)` 将模块注册进 Host 的 `modules` 表，此后这些模块被执行时，`arguments[2]` 就是 Host 的 `__webpack_require__`，从而直接命中 Host 的 `installedModules` 缓存：

```
子应用执行 arguments[2]("react")
    = Host.__webpack_require__("react")
    → installedModules["react"] 已存在（Host 加载 vendors~lib.js 时就缓存好了）
    → 直接返回同一个 react 实例  ✓
```

**单例天然保证**：`installedModules` 是页面级全局对象，只要 Host 先加载过，子应用就永远拿同一份缓存，不会出现两个 react 实例的问题。

---

## 三、环境隔离

没有使用 iframe，靠以下三层机制实现隔离：

### 3.1 App ID + bundleId 模块命名空间

Host 和每个子应用的模块 ID 都加了 `$$app.{bundleId}.{version}/` 前缀（`NamedModulesPlugin` 的 `rootName` 参数控制），避免不同应用的私有模块 ID 冲突：

```
Host 模块：  "$$app.1000019.2/src/main.ts"
子应用模块： "$$app.1003596.154/src/App.tsx"
```

共享依赖则直接用包名作为 ID（无前缀），确保 Host 和子应用能互相找到对方缓存的同一个模块。

### 3.2 CC 组件类名去重（针对 Cocos 场景的核心隔离手段）

`ccpack/src/fixCCClassNameConflict.ts` 处理三种命名冲突，统一追加 `appBundleId` 后缀：

| 冲突类型 | 原始值 | 处理后 |
|---------|--------|--------|
| CC 组件类名 | `"Camera"` | `"Camera_1003596"` |
| `@ccclass` 装饰器参数 | `@ccclass('Camera')` | `@ccclass('Camera_1003596')` |
| 场景 JSON `__type__` | `"Camera"` | `"Camera_1003596"` |

确保不同子应用的 Cocos 组件注册名全局唯一，不互相覆盖。

### 3.3 框架级隔离注册

Host 按条件按需加载不同框架运行时：

```typescript
// tutor-box-cocos-hall/packages/hall/src/main.ts
root
  .registerDelayFramework('cocos-framework',
      ({ currentAppId }) => !currentAppId || +currentAppId >= 10000,
      async () => { /* Cocos 运行时 */ }
  )
  .registerDelayFramework('react-framework',
      ({ webApps }) => webApps?.some(app => app.loadType & AppType.ReactSubApp),
      async () => (await import('@tutor/react-framework')).reactFramework
  )
```

React 子应用通过 `ReactSubgame.register(App)` 挂载，由 Host 控制生命周期，但共享同一个 react 实例。

---

## 四、ccpack 做了什么

ccpack（`~/yuanli/ccpack`）是针对 **Cocos Creator 游戏子应用**的构建后处理工具，主要解决多个 Cocos 子游戏合包时的命名冲突问题。**hall 本身不走 ccpack**，hall 用的是 `@tutor/cocos-compiler`（一个基于 webpack 的完整编译器）。

ccpack 的五个核心变换：

1. **replaceModuleIdToModuleName.ts** — Cocos 原始产物里数字模块 ID → npm 包名
2. **createLibSourceFile.ts** — 抽取共享依赖生成分层产物（lib.js / hall.js）
3. 生成 project.js — 提取子游戏逻辑
4. **fixCCClassNameConflict.ts** — 处理三类 Cocos 类名冲突（见上）
5. 打 zip 包、上传 OSS

常用命令：

```bash
ccpack -dzul    # 构建 hall：关闭压缩、zip、上传，使用本地 hall
ccpack -dszu    # 构建子游戏：dev 模式、分离配置、zip、上传
ccpack -sz      # 仅打包 zip
```

> 注：`ccpack/registerModule.js` 和 `bootstrap.js` 是 ccpack 处理 **Cocos 子游戏**时注入的运行时文件，与 hall + camera-lib 这条 webpack 链路无关。

---

## 五、扩展方案：接入 Vite 子应用

### 5.1 现有方案的限制

现有共享机制的桥是 **webpack 模块调用约定**：webpack 执行每个模块工厂时固定传 `(module, exports, __webpack_require__)`，子应用用 `arguments[2]` 向上取依赖。Vite 生产构建用 Rollup 输出 ESM，没有模块工厂函数这个概念，`arguments[2]` 直接断掉。

### 5.2 方案：显式暴露 `window.__tutor_shared`

将 Host 已初始化的共享依赖暴露到一个**不可篡改的全局命名空间**，让任意构建工具的子应用都能访问。

**Host 初始化阶段（hall 启动时执行一次）：**

```javascript
Object.defineProperty(window, '__tutor_shared', {
    value: Object.freeze({
        'react':                   require('react'),
        'react-dom':               require('react-dom'),
        'rxjs':                    require('rxjs'),
        'mobx':                    require('mobx'),
        '@tutor/box-live-common':  require('@tutor/box-live-common'),
        '@tutor/react-framework':  require('@tutor/react-framework'),
        // ...与 sharedModule 列表保持一致
    }),
    writable: false,      // window.__tutor_shared = xxx → 无效
    configurable: false,  // 二次 defineProperty 改写 → 报错
    enumerable: false,    // Object.keys(window) 不可见
})
```

`Object.defineProperty` + `Object.freeze` 双重锁，堵死四条篡改路径：

| 攻击方式 | 防御 |
|---------|------|
| `window.__tutor_shared = {}` | `writable: false`，静默失败（严格模式报错） |
| `window.__tutor_shared.react = other` | `Object.freeze` 阻止属性修改 |
| `delete window.__tutor_shared.react` | `Object.freeze` 阻止属性删除 |
| `Object.defineProperty(window, '__tutor_shared', {...})` | `configurable: false` 直接报错 |

**webpack 子应用（ExternalsPlugin 改写目标）：**

```javascript
// 原来：arguments[2]("react")
// 改为：
callback(null, `root window.__tutor_shared["${request}"]`)
// 即 import react → window.__tutor_shared["react"]
```

**Vite 子应用（新接入）：**

```javascript
// vite.config.ts
build: {
    rollupOptions: {
        external: ['react', 'rxjs', '@tutor/box-live-common'],
        output: {
            globals: {
                'react':                   `window.__tutor_shared["react"]`,
                'rxjs':                    `window.__tutor_shared["rxjs"]`,
                '@tutor/box-live-common':  `window.__tutor_shared["@tutor/box-live-common"]`,
            }
        }
    }
}
```

### 5.3 与现有方案对比

> 注：`vendors~lib.js` 是 `<script>` 同步加载的，hall 启动时这些模块实际上已全部初始化，两套方案在"加载时机"上没有实质区别。

| | 现有方案（webpack runtime 共享） | window.__tutor_shared |
|--|------|--------|
| 共享模块载体 | webpack `installedModules`（隐式） | `window.__tutor_shared`（显式） |
| 构建工具限制 | **仅 webpack** | **任意** |
| 篡改防护 | webpack 内部封装，外部无法访问 | `Object.defineProperty` + `Object.freeze` 双重锁 |
| 调试 | 隐式，需通过 webpack devtools | 直接 `window.__tutor_shared` 查看 |
| Host 改动 | 不需要 | 启动时加一次暴露 |
| 单例保证 | 同一 runtime 缓存，天然单例 | 同一 window 对象，天然单例 |

**适用场景**：团队有新项目用 Vite 构建并需要接入 hall 体系时，最小改动的可行路径。现有 webpack 子应用和新 Vite 子应用可以并存，Host 侧只需加一次初始化。

---

## 六、与标准 Webpack Module Federation 对比

| 能力 | 这套方案 | Webpack MF |
|------|---------|-----------|
| 共享模块载体 | webpack `installedModules` 缓存 | Webpack 内部 shared scope |
| 单例 | 天然（同一个 runtime 缓存） | 需手动 `singleton: true` |
| 版本协商 | 无，依赖构建时版本对齐 | 运行时 `requiredVersion` 协商 |
| 模块 ID 管理 | `NamedModulesPlugin` 改为包名 | Webpack MF 自动处理 |
| 命名冲突处理 | ccpack 追加 bundleId 后缀 | `publicPath` 隔离 |
| 隔离方式 | 模块 ID 命名空间 + App ID | 独立 webpack runtime |
| 适用场景 | Cocos Creator 多子应用 | 通用 Web 微前端 |

**核心权衡**：复用了 webpack 自身的 chunk 共享能力，不引入额外运行时；代价是所有子应用必须与 Host 使用同一个 webpack runtime，版本必须构建时对齐。

---

## 七、怎么保证最后打包是对的

### 先说结论

整个过程只需要保证三件事：

1. **Host 提供共享依赖**：React 等公共包只由 Host 打包。
2. **Sub 使用 Host 的依赖**：子应用不再打包 React，而是从 Host 获取。
3. **最后做验证**：确认子应用里没有第二份 React，并且双方拿到的是同一个 React 对象。

```text
确定共享依赖 → 打包 Host → 打包 Sub → 先加载 Host → 加载 Sub → 验证
```

### 7.1 确定共享依赖

Host 和 Sub 的共享依赖名称必须对应：

```javascript
// Host
new NamedSharedModulesPlugin(['react', 'react-dom', 'react-dom/client'])

// Sub
const SHARED_MODULES = ['react', 'react-dom/client']
```

注意：`react-dom` 和 `react-dom/client` 是两个不同模块。Sub 使用哪个名称，Host 就必须提供哪个名称。

### 7.2 打包 Host

Host 构建时要做三件事：

- 给共享依赖设置固定模块 ID，例如 `"react"`。
- 把 React 等依赖抽到 `vendors.js`。
- 生成唯一的 `runtime.js`，统一管理模块。

最终得到：

```text
runtime.js   webpack 模块运行时
vendors.js   React 等共享依赖
main.js      Host 业务代码
```

### 7.3 打包 Sub

Sub 构建时要做三件事：

- 把 React 设置为 external，不把 React 源码打进子应用。
- 将 React 引用改为 `__webpack_require__("react")`，使其从 Host 获取。
- 给私有模块增加 `sub-app/` 前缀，避免模块 ID 冲突。

Sub 最终通过 `window.webpackJsonp.push(...)` 把自己的模块注册到 Host。

### 7.4 保证加载顺序

```text
runtime.js → vendors.js → Host main.js → Sub main.js
```

Host 必须先准备好 runtime、共享依赖和子应用容器，然后才能加载 Sub。

### 7.5 验证打包结果

先执行：

```bash
cd /Users/huanghualian/projects/my_pratise/learn-webpack-shared
pnpm build
```

然后检查两个重点：

1. Sub 的 `main.js` 很小，没有包含完整 React 源码。
2. 浏览器控制台执行下面代码，结果必须是 `true`：

```javascript
window.__HOST_REACT__ === window.__SUB_REACT__
```

满足这两个条件，就说明 React 没有重复打包，而且 Host 与 Sub 共用了同一个实例。

> 当前 demo 已实际构建成功；Sub 产物约 2.54 KiB，没有包含完整 React 实现。

---

## 八、关键文件速查

| 文件 | 作用 |
|------|------|
| `tutor-box-cocos-hall/packages/hall/dist/src/runtime.js` | webpack runtime，定义 `__webpack_require__` 和 `webpackJsonp` |
| `tutor-box-cocos-hall/packages/hall/dist/src/vendors~lib.js` | 共享 npm 依赖 chunk |
| `tutor-box-cocos-hall/node_modules/@tutor/cocos-compiler/dist/lib/plugin/named-modules-plugin.js` | 将 sharedModule 的 ID 改为包名 |
| `tutor-box-cocos-hall/packages/hall/ccpack.config.ts` | Host 构建配置，含 sharedModule 列表 |
| `tutor-box-cocos-hall/packages/hall/src/main.ts` | Host 入口，框架注册 |
| `tutor-box-camera-lib/config-overrides.js` | 子应用 webpack 配置，声明 remoteModules |
| `tutor-box-camera-lib/node_modules/@tutor/ccpack-react-plugin/dist/lib/webapp/webpack.js` | ExternalsPlugin，将 remoteModules 改写为 `arguments[2](包名)` |
| `tutor-box-camera-lib/src/index.tsx` | 子应用入口，`ReactSubgame.register(App)` |
| `ccpack/src/fixCCClassNameConflict.ts` | Cocos 子游戏类名冲突解决 |
