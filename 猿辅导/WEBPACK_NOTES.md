# Webpack 学习笔记

## 1) Plugin 是什么？能做什么？

### 1.1 Plugin 的本质

- **Plugin 本质**：一个带 `apply(compiler)` 方法的对象（class 实例也行）
- **工作方式**：在 `apply` 里通过 `compiler.hooks.xxx.tap(...)` / `compilation.hooks.xxx.tap(...)` 把回调挂到构建生命周期钩子上
- **能改什么**：构建流程、module/chunk 的组织方式、输出 assets、watch/HMR 相关流程、统计上报等

### 1.2 compiler vs compilation

- **`compiler`**：一次“整体构建”的全局对象（启动 -> 结束）
- **`compilation`**：一次“编译/构建产物生成”的容器（watch 下会有多次 compilation）

### 1.3 常见 hooks（理解“什么时候触发”）

> 本仓库用 `HookTracePlugin` 打印触发顺序（见下方 1.4）

- **compiler 常见**：`environment`、`beforeRun/run`、`watchRun`、`thisCompilation/compilation`、`make`、`emit/afterEmit`、`done/failed`
- **compilation 常见**：`buildModule/succeedModule/finishModules`、`seal`、`processAssets/afterProcessAssets`


## 2) Loader 是什么？和 Plugin 的区别是什么？

### 2.1 Loader 的本质

- **Loader 本质**：一个函数，处理“单个模块”的源码
- **输入/输出**：`source(string|Buffer)` -> 返回新源码（或通过 callback 异步返回）
- **位置**：在“模块构建（build module）”阶段执行，是 module pipeline 的一部分

### 2.2 Plugin vs Loader（最实用的判断法）

- 你想做的是 **“某类文件内容怎么变”**（逐模块转换）：用 **Loader**
  - 例：TS/JSX 转 JS、CSS 转 JS 模块、对模块做 AST 替换
- 你想做的是 **“构建过程/产物如何组织、何时做事、输出额外东西”**：用 **Plugin**
  - 例：生成 manifest、修改/新增输出 assets、统计耗时/体积、影响拆包 chunk

> 简单说：**Loader 盯“源码/模块”，Plugin 盯“流程/产物”**。

---

## 3) Loader 链的执行顺序：pitch & normal

Loader 有两段可执行逻辑：

- **pitch 阶段**：从左到右（loader1.pitch -> loader2.pitch -> loader3.pitch）
- **normal 阶段**：从右到左（loader3 -> loader2 -> loader1）

本仓库用 `pitch-trace-loader` 打印顺序（运行：`npm run loader:pitch` 或 `npm run loader:all`）。

---

## 4) Webpack 模块化实现 & 动态加载（runtime）

这一节对应的是你在 `dist/bundle.js` 里能直接看到的 runtime 代码，帮助你把“源码 -> bundle -> 动态 chunk”串起来。

### 4.1 模块化的基本结构（IIFE + modules map）

Webpack 默认会把入口 chunk 包一层 **IIFE**（立即执行函数），内部维护一个“模块工厂表”：

- **IIFE 外壳**：隔离作用域，避免全局污染
- **`__webpack_modules__`**：一个对象，key 是模块 id（比如 `"./src/utils.js"`），value 是模块工厂函数

你在产物里能看到类似结构（节选）：

```1:64:webpack-test/dist/bundle.js
/******/ (() => { // webpackBootstrap
/******/ 	"use strict";
/******/ 	var __webpack_modules__ = ({
// ... 每个模块一个工厂函数 ...
/******/ 	});
/******/ 	// The module cache
/******/ 	var __webpack_module_cache__ = {};
/******/ 	
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/ 		// Check if module is in cache
/******/ 		var cachedModule = __webpack_module_cache__[moduleId];
/******/ 		if (cachedModule !== undefined) {
/******/ 			return cachedModule.exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = __webpack_module_cache__[moduleId] = {
/******/ 			exports: {}
/******/ 		};
/******/ 	
/******/ 		// Execute the module function
/******/ 		__webpack_modules__[moduleId](module, module.exports, __webpack_require__);
/******/ 	
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
```

### 4.2 `__webpack_require__` 做了什么（核心就是“执行 + 缓存”）

你总结的点完全对：
- **执行模块工厂函数**：`__webpack_modules__[moduleId](module, module.exports, __webpack_require__)`
- **缓存**：`__webpack_module_cache__`（同一个模块只执行一次，后续直接复用 `exports`）

同时还有一堆 runtime 小工具挂在 `__webpack_require__` 上（比如 ESModule 的导出 getter、publicPath、chunk 文件名等）。

### 4.3 动态 import 的执行链路（JSONP chunk loading）

在 Webpack 5 的浏览器 runtime 下，`import()` 会编译成类似：

```311:318:webpack-test/dist/bundle.js
setTimeout(async () => {
    // 动态导入模块
    const module = await __webpack_require__.e(/*! import() */ "src_dynamic-module_js").then(__webpack_require__.bind(__webpack_require__, /*! ./dynamic-module */ "./src/dynamic-module.js"));
    console.log('模块加载完成:', module);
}, 1000); 
```

对应你整理的步骤，可以对照 `dist/bundle.js` 的 runtime 代码理解：

- **(1) 入口：`__webpack_require__.e(chunkId)`**
  - 作用：确保 chunk 被加载完成（返回 Promise）
  - 内部会遍历 `__webpack_require__.f` 里注册的加载器（当前主要是 `.f.j` 负责 JS chunk）

```82:93:webpack-test/dist/bundle.js
/******/ 	__webpack_require__.f = {};
/******/ 	__webpack_require__.e = (chunkId) => {
/******/ 		return Promise.all(Object.keys(__webpack_require__.f).reduce((promises, key) => {
/******/ 			__webpack_require__.f[key](chunkId, promises);
/******/ 			return promises;
/******/ 		}, []));
/******/ 	};
```

- **(2) 进入 `__webpack_require__.f.j`：用 JSONP 方式加载 JS chunk**
  - 作用：如果 chunk 没加载过，就创建 Promise，拼出 `url = publicPath + chunkFilename`，然后注入 `<script src="...">`

```201:249:webpack-test/dist/bundle.js
/******/ 	var installedChunks = { "main": 0 };
/******/ 	__webpack_require__.f.j = (chunkId, promises) => {
/******/ 		var installedChunkData = __webpack_require__.o(installedChunks, chunkId) ? installedChunks[chunkId] : undefined;
/******/ 		if(installedChunkData !== 0) {
/******/ 			if(installedChunkData) {
/******/ 				promises.push(installedChunkData[2]);
/******/ 			} else {
/******/ 				var promise = new Promise((resolve, reject) => (installedChunkData = installedChunks[chunkId] = [resolve, reject]));
/******/ 				promises.push(installedChunkData[2] = promise);
/******/ 				var url = __webpack_require__.p + __webpack_require__.u(chunkId);
/******/ 				__webpack_require__.l(url, loadingEnded, "chunk-" + chunkId, chunkId);
/******/ 			}
/******/ 		}
/******/ 	};
```

- **(3) chunk 加载完成后：执行 `self["webpackChunkxxx"].push(...)`**
  - 作用：把“这个 chunk 新增的模块工厂”塞回主 bundle 的 `__webpack_require__.m` 里
  - push 的回调就是 runtime 里安装的 `webpackJsonpCallback`

```261:289:webpack-test/dist/bundle.js
/******/ 	var webpackJsonpCallback = (parentChunkLoadingFunction, data) => {
/******/ 		var [chunkIds, moreModules, runtime] = data;
/******/ 		if(chunkIds.some((id) => (installedChunks[id] !== 0))) {
/******/ 			for(moduleId in moreModules) {
/******/ 				__webpack_require__.m[moduleId] = moreModules[moduleId];
/******/ 			}
/******/ 			if(runtime) var result = runtime(__webpack_require__);
/******/ 		}
/******/ 		for(;i < chunkIds.length; i++) {
/******/ 			if(__webpack_require__.o(installedChunks, chunkId) && installedChunks[chunkId]) {
/******/ 				installedChunks[chunkId][0](); // resolve()
/******/ 			}
/******/ 			installedChunks[chunkId] = 0;
/******/ 		}
/******/ 	}
/******/ 	var chunkLoadingGlobal = self["webpackChunkwebpack_test"] = self["webpackChunkwebpack_test"] || [];
/******/ 	chunkLoadingGlobal.push = webpackJsonpCallback.bind(null, chunkLoadingGlobal.push.bind(chunkLoadingGlobal));
```

- **(4) Promise resolve 后：`.then(__webpack_require__(/*! moduleId */))` 真正取到模块**
  - 这就是为什么你会看到 `__webpack_require__.e(...).then(__webpack_require__.bind(...))` 这段生成代码：先确保 chunk 到位，再 require chunk 里那个模块 id。

> 备注：你提到的 “HMR/热更新” 也是 runtime 的一部分，但它需要 dev-server/hot runtime 支持；你当前这个 `webpack-test` 的产物里能看到 `// no HMR` 的注释（说明没开启）。

## 5) Tapable 是什么？怎么用？

### 5.1 Tapable 的定位

**Tapable 是 Webpack 的 Hook 系统实现**（发布/订阅 + 可控执行模型）。

- Webpack 内部：创建各种 Hook（Sync/Async/Waterfall/Bail/Parallel…）
- Webpack 执行流程：在关键节点 `hook.call(...) / hook.promise(...)`
- Plugin：用 `tap/tapAsync/tapPromise` 订阅

### 5.2 你在 Plugin 里用到的就是 Tapable

例如：
- `compiler.hooks.done.tap(...)`
- `compilation.hooks.processAssets.tap(...)`

这些 `hooks` 本质都是 Tapable 的 hook 实例。

### 5.3 常见 Hook 类型（理解“顺序/并行/短路”）

- `SyncHook`：同步串行
- `SyncBailHook`：同步串行，某个回调返回非 `undefined` 就“短路停止”
- `SyncWaterfallHook`：同步串行，上一个回调返回值传给下一个
- `AsyncSeriesHook`：异步串行（一个完成再下一个）
- `AsyncParallelHook`：异步并行（一起跑，等都完成）

---

## 6) VSCode 调试小贴士（你之前遇到的坑）

你之前遇到过：VSCode 启动调试时用旧 Node（v14）导致 webpack-cli 报 `Cannot find module 'node:events'`。

做法：
- 保证 VSCode 运行时使用 **Node 18+/20+**
- 或在 `.vscode/launch.json` 里指定 `runtimeExecutable` 到正确的 `npm/node`


