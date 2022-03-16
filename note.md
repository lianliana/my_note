## CSS

### CSS选择器
- 1000：内联样式
- 100： id选择器
- 10：类选择器、伪类选择器、属性选择器
- 1：标签选择器、伪元素选择器
！import的优先级最高
- 样式表的来源不同时，优先级顺序为：内联样式 > 内部样式 > 外部样式 > 浏览器用户自定义样式 > 浏览器默认样式。

### CSS中可继承和不可继承属性
#### 可继承
1. display：
2. 盒子模型相关属性
3. 背景属性
4. 定位属性
5. 文本属性
6. 生成内容属性
7. 页面样式属性
#### 不可继承
1. 字体的属性
2. 元素可见性
3. 列表布局属性
4. 光标属性

### display的属性有哪些
1. none
2. block ：默认宽度为父元素宽度 可设置宽高，换行显示
3. inline-block ：默认宽度为内容宽度 可设置宽高 同行显示
4. flex
5. table 此元素会作为块级表格来显示
6. inline 默认宽度为内容宽度 不可设置宽高 同行显示
7. list-item

### 隐藏元素的方法有哪些
1. display:none 不会渲染 也不会响应事件 也不占据位置
2. visibility:hidden 仍占据位置 不响应事件
3. opacity:0 透明度为0，只是看不见 其他都响应
4. postion：abosolute 脱离文档流 挪到别的地方
5. z-index 让其他元素覆盖
6. transform: scale(0,0) 将元素缩放为0

### link和@import 区别
- link 是xhtml标签 可以加载CSS之外的资源 并且引入时同时加载 无兼容性问题
- @import 是CSS提出的  会有兼容性问题 只能引入CSS

### transition和animation的区别
- transition是过度动画 需要指定触发事件
- animation是动画属性 不需要触发 设置好时间执行

### 如何判断元素是否到达可视区域
- window.innerHeight 可视高度
- document.body.scrollTop || document.documentElement.scrollTop 是浏览器滚动过的距离
- imgs.offsetTop 是元素顶部距离文档顶部的高度

所以文档到达可视区域的时候是通过 imgs.offsetTop<window.innerHeight+document.scrollTop 

### flex中的属性
#### 容器中属性
- flex-direction 决定主轴的方向
- flex-wrap 如何换行
- flex-flow 以上两个的简写形式
- justify-content 主轴方向上的对齐方式
- align-items 交叉轴上如何对齐
#### 项目中属性
- order 项目排列顺序 默认为0 越小越前
- flex-grow 项目放大比例 默认为0
- flex-shrink 项目缩小比例 默认为1
- flex-basis 分配多余空间时占据多少 默认auto
### 创建BFC的条件：
- 根元素：body；
- 元素设置浮动：float 除 none 以外的值；
- 元素设置绝对定位：position (absolute、fixed)；
- display 值为：inline-block、table-cell、table-caption、flex等；
- overflow 值为：hidden、auto、scroll；


