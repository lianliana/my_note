# React 学习总结

> React 应用层与版本演进相关的总结。Fiber 内核源码相关的内容见 `ReactFiber学习总结.md`。

---

## 目录

- [1. React 各版本演进对比](#1-react-各版本演进对比)
- [2. useEvent / useMemoizedFn 模式：稳定引用 + 最新闭包](#2-useevent--usememoizedfn-模式稳定引用--最新闭包)

---

## 1. React 各版本演进对比

> 从 React 15 到 React 19，每一代的核心变化与对应用层的实际影响。

### 1.1 总览表

| 版本 | 发布时间 | 架构关键词 | 应用层关键能力 |
|---|---|---|---|
| **React 15** | 2016 | Stack Reconciler（递归、同步、不可中断） | `createClass`、ReactDOM 拆出 |
| **React 16** | 2017 | **Fiber 架构**、双缓冲、可中断基础设施 | Error Boundaries、Fragments、Portals、新生命周期 |
| **React 16.8** | 2019 | （架构沿用 16） | **Hooks 正式发布** |
| **React 17** | 2020 | "无新特性"垫脚石 | 事件委托从 document 迁到 root；新 JSX Transform；逐步升级 |
| **React 18** | 2022 | **Concurrent Rendering**、Lanes 模型、Flags 替代 effectList | `createRoot`、Automatic Batching、Transitions、Suspense for Data、Streaming SSR、5 个新 hooks |
| **React 19** | 2024 | React Compiler（自动 memo）、Server Components 稳定 | Actions、`use`、`useOptimistic`、`useActionState`、ref as prop、Document Metadata |

### 1.2 React 15 → 16：Stack Reconciler 到 Fiber

#### Stack Reconciler 的问题

- 递归调和整棵树，**一旦开始必须跑完**
- 大组件树更新时主线程被长时间占用，导致掉帧、输入卡顿
- 浏览器无法插入高优先级任务（动画、用户交互）

#### Fiber 解决了什么

- 把"递归调用栈"改成**链表 + 循环**：每个 Fiber 是一个工作单元，可以在两次任务之间被打断
- **双缓冲**：`current` 树（已提交）与 `workInProgress` 树（构建中）并存，构建被打断也不影响 UI
- **优先级调度**（雏形，叫 `expirationTime`）：高优更新可以打断低优渲染
- 把更新过程切成 **render 阶段（可中断）+ commit 阶段（不可中断）**

> **重要：Fiber 只是基础设施**。16 默认仍是同步行为，"真正能用上中断"要等到 18 的 Concurrent 模式。

#### 同时引入的应用层特性

- **Error Boundaries**（`componentDidCatch`）：捕获子树渲染错误
- **Fragments**（`<></>`）：组件可返回数组
- **Portals**（`createPortal`）：把子树渲染到 DOM 外部位置
- **新生命周期**：`getDerivedStateFromProps` / `getSnapshotBeforeUpdate`；废弃 `componentWillMount` 等"不安全"生命周期（为 Concurrent 铺路）

### 1.3 React 16.8：Hooks

- 函数组件第一次获得"持有状态"的能力
- 核心 hooks：`useState` / `useEffect` / `useRef` / `useMemo` / `useCallback` / `useContext` / `useReducer` / `useLayoutEffect` / `useImperativeHandle` / `useDebugValue`
- 底层实现：函数组件 Fiber 上的 `memoizedState` 维护一条 **Hook 单链表**（详见 `ReactFiber学习总结.md` 第 6 节）
- 强制约束：**调用顺序必须一致**（不能在条件/循环里调）

> 现在的 React 业务代码几乎都基于 Hooks，类组件进入"维护期"。

### 1.4 React 17：垫脚石版本（几乎无新 API）

虽然版本号 +1，但**几乎没有新特性**，专门为"渐进式升级"做底层调整：

| 变化 | 原因 |
|---|---|
| 事件委托：`document` → React root 节点 | 解决多版本 React 共存的事件冲突，方便微前端/局部升级 |
| 新 JSX Transform（不再需要 `import React from 'react'`） | 编译器直接生成 `react/jsx-runtime` 调用，包体积更小 |
| `useEffect` cleanup 改为异步执行 | 与浏览器绘制对齐，性能更好 |
| 移除"事件池"（event pooling） | 让事件对象语义更符合直觉 |

> 一句话：**17 不是为了用户，是为了让你能放心升 18。**

### 1.5 React 18：Concurrent Rendering 正式落地

最大的一代变化，所有 Concurrent Features **必须用 `createRoot` 才启用**：

```tsx
import { createRoot } from 'react-dom/client'
createRoot(container).render(<App />)
```

#### 调度/渲染层

| 特性 | 说明 |
|---|---|
| **Lanes 模型** | 取代旧 `expirationTime`，用 31 位 bitmask 表达"优先级车道"，支持并发处理多个优先级 |
| **Flags 取代 effectList** | 副作用标记下沉到 Fiber 自身的 `flags` + `subtreeFlags`，commit 阶段按"子树位"决定是否进入。删除了 `firstEffect/nextEffect` 链表 |
| **Automatic Batching** | setTimeout / Promise / 原生事件回调里的多个 setState 也会自动批处理（17 只在 React 事件里批） |
| **Transitions** | `startTransition` / `useTransition` 标记非紧急更新，可被中断 |
| **Suspense 增强** | 支持服务端、数据请求场景；fallback 显示更稳定 |
| **Streaming SSR** | `renderToPipeableStream`，HTML 流式输出 + Selective Hydration |

#### 新增 hooks

| Hook | 作用 |
|---|---|
| `useId` | 生成跨服务端/客户端稳定的唯一 id（解决 SSR 水合 id 冲突） |
| `useTransition` | 标记一组 state 更新为低优先级（不阻塞 UI） |
| `useDeferredValue` | 让某个值"延迟跟上"，配合搜索框这种场景 |
| `useSyncExternalStore` | 给外部数据源（如 Redux）接入 React 并发渲染的官方 API |
| `useInsertionEffect` | 比 useLayoutEffect 更早执行，CSS-in-JS 库专用（普通业务别用） |

#### Strict Mode 增强

- 开发模式下，组件会被**故意 mount → unmount → mount**，验证你的代码能否承受重复挂载（为未来的 Offscreen API 铺路）
- Effect 会被故意双调用，强化对副作用清理的检查

### 1.6 React 19：Compiler + Actions + Server Components

#### React Compiler（自动 memo）

- 编译器在构建期自动给组件、回调、值添加 memoization
- **理论上不再需要手写 `useMemo` / `useCallback` / `React.memo`**
- 当前为可选（opt-in），通过 babel 插件启用
- 这意味着第 2 节讨论的 `useEvent` / `useMemoizedFn` 模式，未来可能也被编译器消化掉

#### Actions（表单 / 异步状态）

```tsx
function Form() {
    const [error, submitAction, isPending] = useActionState(
        async (_, formData) => {
            try { await save(formData); return null }
            catch (e) { return e.message }
        },
        null
    )
    return <form action={submitAction}>...</form>
}
```

| 新 API | 作用 |
|---|---|
| `<form action={fn}>` | form 元素可直接绑定 async 函数 |
| `useActionState` | 管理 action 的 pending / error / data 状态 |
| `useFormStatus` | 表单子组件获取当前提交状态 |
| `useOptimistic` | 乐观更新（提交时立刻显示预期结果） |

#### `use` —— 可条件调用的 Hook

```tsx
const data = use(promise)
const theme = use(ThemeContext)
```

- 这是 React 第一个**允许条件调用**的 hook（打破"只能顶层调用"规则）
- 让条件读 Context 成为可能

#### ref 不再需要 forwardRef

```tsx
function MyInput({ ref, ...props }) {
    return <input ref={ref} {...props} />
}
```

老写法 `React.forwardRef((props, ref) => ...)` 不再必要。

#### Server Components（RSC）稳定

- 一类只在服务端运行、零客户端 JS 的组件
- 可以直接 `await` 数据库 / 文件系统
- 配合 Next.js App Router 已经在生产可用

#### 其他

- **Document Metadata 自动 hoist**：组件里写 `<title>` / `<meta>` 会自动提到 `<head>`
- **资源预加载 API**：`preload` / `preconnect` / `preinit`
- **错误处理改进**：默认不再向 console 输出两份错误堆栈

### 1.7 升级路径建议

| 起点 | 终点 | 关键关注点 |
|---|---|---|
| 15 → 16 | 检查废弃的生命周期，迁移到新的 `getDerivedStateFromProps` 等 |
| 16 → 17 | 几乎无破坏；注意事件委托位置变化（影响 e.stopPropagation 行为） |
| 17 → 18 | `ReactDOM.render` → `createRoot`；检查 Strict Mode 双调用副作用；外部 store 改用 `useSyncExternalStore` |
| 18 → 19 | ref 写法可简化；评估 React Compiler；新 Actions API 替代手写表单状态 |

### 1.8 主线一句话

> **15 是 React 的"成熟期"，16 是架构革命的"播种期"，17 是迁移工具的"工具期"，18 是并发能力的"收获期"，19 是编译时优化与全栈整合的"自动化期"。**

> 主线是：**让 React 从「同步递归渲染库」变成「能感知优先级、可中断、可恢复、能跨端协作」的并发 UI 运行时**。

---

## 2. useEvent / useMemoizedFn 模式：稳定引用 + 最新闭包

> 源自项目 `packages/student-chat-utils/src/hooks/useEvent.ts` 的讨论。

### 2.1 `useCallback` 的两难

```typescript
const fn = useCallback(() => doSomething(state), [state])
```

- 想要闭包最新 → 必须把 `state` 写进依赖 → 引用必然变 → 下游 memo 子组件/effect 反复重跑
- 想要引用稳定 → 不写依赖 → 闭包僵在第一次渲染 → 出现"闭包陷阱"

「引用稳定」和「闭包最新」在 `useCallback` 里是**互斥**的。

### 2.2 useEvent 的实现拆解

```typescript
export function useEvent<T extends (...args: any[]) => any>(callback: T): T {
    const fnRef = React.useRef<any>()                                    // ①
    fnRef.current = callback                                              // ②
    const memoFn = React.useCallback<T>(
        ((...args: any) => fnRef.current?.(...args)) as any,              // ③
        []                                                                // ④
    )
    return memoFn
}
```

| 步骤 | 作用 |
|---|---|
| ① `useRef` | 创建一个跨渲染共享的"盒子"（盒子本身永远是同一个引用） |
| ② 每次渲染赋值 | 把"本次渲染拿到的最新 callback"塞进盒子，**不触发重渲染** |
| ③④ 空依赖的 useCallback | 返回一个"转发器"：自身引用永远稳定，调用时去读 `fnRef.current` |

工作流：

```
外部 ──调用──> memoFn (永远是同一个) ──读取──> fnRef.current (每次渲染被覆盖) ──执行──> 最新闭包
```

### 2.3 为什么必须有 ③④（不能直接 `return fnRef.current`）

如果只有 ①②、直接返回 `fnRef.current`：

- 用户每次渲染传进来的 callback 都是新函数
- 直接返回它 → 外部拿到的引用每次都变
- "引用稳定"这个核心收益就丢了

③④ 用一个 stable 的"壳函数"把"最新 callback"包起来，外部抓的是壳，壳内部去查最新闭包。

### 2.4 关键认知：`ref.current` 不会触发重渲染

| 东西 | 跨渲染是否相同 | 修改后是否触发重渲染 |
|---|---|---|
| `useRef()` 返回的对象 | ✅ 永远相同 | —（也不会去改它） |
| `ref.current` | ❌ 可任意覆盖 | ❌ 静默更新 |
| `useState` 的 setter | — | ✅ 触发重渲染 |

正因为 `ref.current` 是"**静默更新**"的，才能在渲染期间安心赋值而不形成死循环。"最新闭包"是**懒读取**，等下次有人调用 `memoFn()` 时才拿。

### 2.5 和 ahooks `useMemoizedFn` 的差异

ahooks 源码（核心部分）：

```typescript
const fnRef = useRef(fn)
fnRef.current = useMemo(() => fn, [fn])              // ② 改进
const memoizedFn = useRef()
if (!memoizedFn.current) {                            // ③ 改进
    memoizedFn.current = function (this, ...args) {   // ④ 改进
        return fnRef.current.apply(this, args)
    }
}
return memoizedFn.current
```

| 维度 | useEvent | useMemoizedFn |
|---|---|---|
| 核心思想 | ref 存最新闭包 + 稳定壳函数 | 一样 |
| 返回引用稳定性 | ✅ | ✅ |
| 闭包始终最新 | ✅ | ✅ |
| 类型校验 | ❌ | ✅ 开发期 warning |
| Concurrent 安全 | ⚠️ 理论有竞态 | ✅ `useMemo` 修复 |
| 壳函数实现 | `useCallback([])` | `useRef + if` |
| 保留 `this` | ❌ 箭头函数 | ✅ `function + apply` |

**关键差异点**：

1. **`useMemo` vs 直接赋值**（最重要）
   - React 18 Concurrent 模式下，渲染可能被丢弃，直接赋值会让"被丢弃的渲染"污染 ref
   - `useMemo` 的回调结果与提交阶段一致，被丢弃的渲染不会泄露
   - 对应 issue：[alibaba/hooks#728](https://github.com/alibaba/hooks/issues/728)

2. **`function + apply` vs 箭头函数**
   - 箭头函数：`this` 永远是 `undefined`，传给 DOM 事件 / 类组件方法时 `this` 丢失
   - `function + apply`：转发 `this`，更通用
   - 业务里 99% 用箭头函数 + 闭包，碰不到差异

### 2.6 useEvent 的适用边界（不是 useCallback 超集！）

#### 翻车场景 1：渲染期间调用

```tsx
const process = useEvent(() => data.process())
const result = process()   // ❌ 拿到的可能是上一帧的 callback
```

`fnRef.current = callback` 这行本身在渲染期间执行，**在它执行之前**调用 useEvent 返回的函数，读到的是上次渲染的旧值。React 18 严格模式双渲染、Concurrent 渲染丢弃也会让这个问题更不可控。

> React 官方对 `useEffectEvent` 的硬性限制：**只能在事件回调或 Effect 里调用，不能在渲染期间调用**。

#### 翻车场景 2：作为 useEffect 依赖时 effect 不会重跑

```tsx
const fetchData = useEvent(() => api.search(query))

useEffect(() => {
    fetchData()
}, [fetchData])   // ❌ fetchData 引用永远稳定，effect 只跑一次
```

`useCallback` 的"依赖变就换引用"特性是**功能性信号**（告诉外界"我变了，你要响应"），不只是性能优化。`useEvent` 把这个信号阉割了。

正确写法：

```tsx
const fetchData = useCallback(() => api.search(query), [query])
useEffect(() => { fetchData() }, [fetchData])   // query 变 → effect 重跑 ✅
```

#### 翻车场景 3：派生函数不是事件

```tsx
const formatter = useCallback(
    (n) => n.toLocaleString(locale, { currency }),
    [locale, currency]
)
return items.map(item => <div>{formatter(item.price)}</div>)
```

这类"派生函数"本质是**数据**不是**事件**，用 `useEvent` 既违反渲染期间不能调用，又错误地把"行为变化"隐藏起来。

### 2.7 决策表

| 场景 | 推荐 | 原因 |
|---|---|---|
| 事件回调（onClick / 异步回调 / 订阅回调） | ✅ `useEvent` | 引用稳定 + 闭包新 |
| 传给 `React.memo` 子组件、子组件只调用不订阅 | ✅ `useEvent` | 避免无意义重渲染 |
| 渲染期间会被调用的派生函数 | ❌ `useEvent`<br>✅ `useCallback` / `useMemo` | useEvent 不能渲染期间调用 |
| useEffect 依赖项，且希望 effect 跟随输入变化 | ❌ `useEvent`<br>✅ `useCallback` | useEvent 让 effect 不重跑 |
| useEffect 依赖项，但希望 effect **不**因为它重跑 | ✅ `useEvent` | 正是 useEffectEvent 设计场景 |
| 暴露给外部的库 API | ⚠️ 谨慎用 `useEvent` | 消费方可能依赖"引用变=行为变"语义 |

### 2.8 一句话区分

> **`useCallback`** 表达 "我是一个**值**，依赖变了我也变"  
> **`useEvent`** 表达 "我是一个**动作的入口**，永远是我，但每次做的事是当下版本"

把"动作入口"和"派生值"搞混，就会出 bug。
