### 相关资料

Prose mirror 和富文本理念入门 https://zhuanlan.zhihu.com/p/123341288

Prose-mirror 中文文档 https://www.xheldon.com/tech/prosemirror-guide-chinese.html

Svg 资料 https://jirengu.github.io/svg-you-should-know/zh-cn/

### cyber-slide-editor 结构

- cse-editor-canvas 提供对外的一个画布

  - editable-block.component 提供一个可编辑的block节点

    - custom-editor-box 使用promisemirror的构造出来的一个个box

    - 其中包括这些box类型![img](https://api2.mubu.com/v3/document_image/06745ebd-95b9-4c0e-abae-d6a48191af8c.png)

    - 其中svg特别强大 可以通过去计算出point 用多边形去画各种形状

  - moveControl去控制移动

  - resizeComponet去控制缩放

### prose mirror

- 核心 f(state) === view 

  - schema 如何去解析

  - view 如何展示

  - state 当前状态数据

  - transaction 如何变化数据