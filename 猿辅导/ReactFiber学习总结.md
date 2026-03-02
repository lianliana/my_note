# React Fiber / Hooks 学习总结

> 基于我们这次阅读 `react-reconciler`（commit：`1fb18e22ae66f...`）的讨论整理。

---

### 1. Fiber 是什么？为什么需要 `tag`（Fiber 类型）

- **Fiber**：React 在 reconciler 内部用于表示“工作单元/节点”的数据结构；一棵 React 树会对应一棵 Fiber 树。
- **`tag`（WorkTag）**：用来标记 Fiber 代表哪一类“节点/组件”，从而在 `beginWork`/`completeWork` 等阶段走不同逻辑分支。

常见 `tag`（直观理解）：
- **`FunctionComponent`**：函数组件（执行函数得到 children）
- **`ClassComponent`**：类组件
- **`HostComponent`**：宿主环境元素节点（DOM 渲染器里是 `<div>`、`<span>` 等）
- **`HostText`**：宿主环境文本节点（DOM 文本）
- **`HostRoot`**：根
- **`Fragment` / `ForwardRef` / `MemoComponent` / `SuspenseComponent`**：对应 React 语法/能力的内部节点

---

### 2. `HostComponent` 是什么？

在 **React DOM** 渲染器里：
- 你写的 `<div />`、`<span />` 等 **原生标签** 对应的 Fiber，就是 **`HostComponent`**。
- `HostComponent` 的 `type` 通常是字符串（如 `'div'`），props 是 `pendingProps/memoizedProps`。

对比：
- **`FunctionComponent`**：没有真实 DOM 实例；它在 render 阶段会“执行组件函数”，算出下一层 children。
- **`HostComponent`**：对应真实宿主实例（DOM element），会在后续阶段创建/更新该实例。

---

### 3. render 阶段 vs commit 阶段：分别做什么？

可以把一次更新拆成两大段：

#### 3.1 render 阶段（可中断/可重试）

目标：**计算出下一棵 Fiber 树，以及“需要在提交时做哪些副作用”**。

主要包含：
- **`beginWork`（递）**：根据 Fiber 类型计算子节点（reconcile children）
  - 函数组件：执行组件（含 hooks），得到 children，再 reconcile
  - HostComponent：根据 children reconcile 出子 Fiber
- **`completeWork`（归）**：为 Host 节点准备提交材料
  - 可能会 **创建宿主实例**（例如创建 DOM element）
  - 可能会把“子宿主节点”**先拼装到父宿主实例上**（`appendAllChildren`）
  - 会计算 “更新 payload”、并通过 `effectTag` 标记需要在 commit 执行的动作（Update/Ref/Placement…）

> 注意：render 阶段即使创建了 DOM 实例，也只是“在内存里准备好”，**不等于已经插入真实页面**。

#### 3.2 commit 阶段（不可中断，真正改外部世界）

目标：**把 render 阶段算出来的变更一次性提交到宿主环境**。

commit 会做的事不仅是“append 根”，还包括：
- 插入（Placement：insert/append 到正确位置）
- 更新（Update：commitUpdate、commitTextUpdate）
- 删除（Deletion：removeChild + unmount/ref 清理）
- Ref 设置/清理
- 生命周期与 hooks effect（layout/passive）的卸载/安装

---

### 4. `effectTag` / `effectList` 是什么？为什么需要 effectList？

#### 4.1 `effectTag`（Fiber 上的 bitmask 标记）

每个 Fiber 有一个 `effectTag`（位掩码），表示 commit 阶段对该 Fiber 要做哪些副作用，比如：
- `Placement`：需要插入/移动
- `Update`：需要更新宿主实例
- `Deletion`：需要删除
- `Ref`：需要处理 ref
- `Passive`：包含 useEffect（被动副作用）相关

#### 4.2 `effectList`（`firstEffect/lastEffect/nextEffect` 单链表）

问题：commit 阶段如果为了找“有哪些 Fiber 有副作用”，再遍历整棵树会很慢。

解决：在 render 阶段的归过程（`completeUnitOfWork`）里，把有副作用的 Fiber **串成一条链表**：
- `fiber.firstEffect`：子树里第一个带 effect 的 Fiber
- `fiber.lastEffect`：子树里最后一个带 effect 的 Fiber
- `fiber.nextEffect`：effectList 的 next 指针

最终 rootFiber 上会拿到一条从 `root.firstEffect` 开始的单向链表，commit 阶段 **线性遍历 effectList** 即可处理所有副作用。

#### 4.3 mount 也需要 effectList

不是只有 update 才需要：
- mount 时也要做 Placement、Ref、生命周期、useEffect/useLayoutEffect 等
- update 时更明显，因为会混合 Update/Deletion/Placement(移动)/effect destroy&create 等

---

### 5. 一个直观例子：`<div><Child />hello</div>` 的首次 mount 怎么“挂上去”？

```jsx
function Child() {
  return <span>hi</span>;
}

export default function App() {
  return (
    <div>
      <Child />
      hello
    </div>
  );
}
```

粗略的 Fiber 形态：
- `HostRoot`
  - `HostComponent('div')`
    - `FunctionComponent(Child)`
      - `HostComponent('span')`
        - `HostText("hi")`
    - （文本 `"hello"` 可能被宿主层做 direct-text 优化，不一定单独建 HostText fiber，视实现路径而定）

流程要点：
- render/beginWork：`div` reconcile 出 `Child`；`Child` 执行函数得到 `<span>` 子树
- render/completeWork：创建 `<div>` 的 DOM 实例；把子树里终端 Host 节点（`span`/text）append 到 div 实例上
- commit：根据 effectList 执行 Placement，把 `div`（以及其内部已经拼好的子树）插入容器

关键点：**`Child` 自己不是 DOM，它只是产出 DOM 子树的中间层**。

---

### 6. Hooks：state hook vs effect hook 挂在哪？为什么要保证顺序？

#### 6.1 state/memo/ref 等 Hook：挂在 `fiber.memoizedState`（Hook 单链表）

函数组件 Fiber 上的 `memoizedState` 保存一条 **Hook 单向链表**。
- mount：每调用一个 hook，就创建一个 Hook 节点追加到链尾（O(1)，靠 `workInProgressHook` 指针）
- update：按调用顺序从 `currentHook.next` 逐个取旧 Hook，对齐并产生新 Hook

#### 6.2 effect（useEffect/useLayoutEffect）：两层结构

useEffect 相关的信息有两层：
- **它作为 Hook 节点**：同样占据 `memoizedState` Hook 链表中的一个位置（保证“第 N 个 hook”可对齐）
- **它的 Effect 列表**：同时会被 push 到 `fiber.updateQueue.lastEffect` 维护的一条 **effect 环形链表**，供 commit 快速遍历执行

#### 6.3 为什么 effect 列表用环形？

React 这里仅保存 `lastEffect`（尾指针）。
用环形链表可以做到：
- O(1) 追加
- O(1) 拿到头：`firstEffect = lastEffect.next`
- 遍历时天然回到起点结束（do/while）

不用环形当然也能实现，但通常需要额外存 `firstEffect`（head）或引入更多分支处理。

#### 6.4 为什么 Hooks 必须保证调用顺序一致？

因为 React 识别“这是哪个 hook”靠的是 **位置（第几个）**，不是名字。

如果你在 update 时改变了调用顺序（比如把 hook 放进 if/for），就会导致“第 N 个 hook”对齐到错误的旧节点，从而：
- `useState` 读到了原本属于 `useEffect` 的 hook 位
- 或反过来 effect 读到了 state hook 位

因此 Hooks 规则要求：
- 只能在组件顶层调用
- 不能放在条件/循环/嵌套函数里
- 每次渲染调用顺序必须一致

---

### 7. 你可以如何继续深入（可选路线）

- 从 `ReactFiberWorkLoop.new.js` 的 `performUnitOfWork -> completeUnitOfWork` 走一遍，观察 effectList 如何被拼接
- 看 `ReactFiberCompleteWork.new.js` 的 `HostComponent` 分支：创建实例、append children、`markUpdate/markRef`
- 看 `ReactFiberCommitWork.new.js`：`commitPlacement/commitWork/commitHookEffectListMount` 如何真正触发 DOM 与 hooks effect

