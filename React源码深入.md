## React源码深入

我们认为，React 是用 JavaScript 构建**快速响应**的大型 Web 应用程序的首选方式。



#### 两个瓶颈

- CPU的瓶颈
- IO的瓶颈

而落实到实现上，则需要将**同步的更新**变为**可中断的异步更新**。



#### React15架构

React15架构可以分为两层：

- Reconciler（协调器）—— 负责找出变化的组件
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上

##### 缺点

在**Reconciler**中，`mount`的组件会调用[mountComponent (opens new window)](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L498)，`update`的组件会调用[updateComponent (opens new window)](https://github.com/facebook/react/blob/15-stable/src/renderers/dom/shared/ReactDOMComponent.js#L877)。这两个方法都会递归更新子组件。





#### React16架构

React16架构可以分为三层：

- Scheduler（调度器）—— 调度任务的优先级，高优任务优先进入**Reconciler**
- Reconciler（协调器）—— 负责找出变化的组件
- Renderer（渲染器）—— 负责将变化的组件渲染到页面上



#### 代数效应与Generator

 从`React15`到`React16`，协调器（`Reconciler`）重构的一大目的是：将老的`同步更新`的架构变为`异步可中断更新`。

`异步可中断更新` 可以理解为：`更新`在执行过程中可能会被打断（浏览器时间分片用尽或有更高优任务插队），当可以继续执行时恢复之前执行的中间状态。

####  代数效应与Fiber

`React Fiber`可以理解为：

`React`内部实现的一套状态更新机制。支持任务不同`优先级`，可中断与恢复，并且恢复后可以复用之前的`中间状态`。



### Fiber的起源

在`React15`及以前，`Reconciler`采用递归的方式创建虚拟DOM，递归过程是不能中断的。如果组件树的层级很深，递归会占用线程很多时间，造成卡顿。

为了解决这个问题，`React16`将**递归的无法中断的更新**重构为**异步的可中断更新**，由于曾经用于递归的**虚拟DOM**数据结构已经无法满足需要。于是，全新的`Fiber`架构应运而生。

### 总结

- `Reconciler`工作的阶段被称为`render`阶段。因为在该阶段会调用组件的`render`方法。
- `Renderer`工作的阶段被称为`commit`阶段。就像你完成一个需求的编码后执行`git commit`提交代码。`commit`阶段会把`render`阶段提交的信息渲染在页面上。
- `render`与`commit`阶段统称为`work`，即`React`在工作中。相对应的，如果任务正在`Scheduler`内调度，就不属于`work`





## React 性能优化

	1. 通过webpack减小代码体积 (tree shaking、include exclude、 按需引入)
	1. 路由懒加载 提高页面加载速度
	1. 受控组件颗粒化、通过memo  小技巧：**如果组件需要相应的数据才存放在state中 否则可以存放在class类组件的this实例上** 同第4条
	1. 善用缓存，可以将不用驱动组件更新的数据绑定在实例this上
	1. 合理正确使用key去使diff算法尽可能命中 减少增加和删除节点的开销
	1. 绑定事件尽量不要用箭头函数，因为每次更新的时候都会创建实例函数
	1. 通过shouldComponentUpdate、PureComponent、React.memo来控制react的渲染 (因为react 在render的时候牵一发而动全身)
	1. 
