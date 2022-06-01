## HTML

### 行内元素、块级元素

块级元素：div ul table form ol  audio video Footer

行内元素：a span img strong  buttom input select label 

空元素：br hr img input link meta

### 行内元素与块级元素的区别？

```
HTML4中，元素被分成两大类：inline （内联元素）与 block （块级元素）。

（1） 格式上，默认情况下，行内元素不会以新行开始，而块级元素会新起一行。
（2） 内容上，默认情况下，行内元素只能包含文本和其他行内元素。而块级元素可以包含行内元素和其他块级元素。
（3） 行内元素与块级元素属性的不同，主要是盒模型属性上：行内元素设置 width 无效，height 无效（可以设置 line-hei
     ght），设置 margin 和 padding 的上下不会对其他元素产生影响。
```



### HTML5 有哪些新特性、移除了那些元素？

新增： 

1. WebWorker、Websocket
2. 一些语义化的标签 Footer Header
3. canvas画布 
4. localStorage和sessionStorage
5. 用于媒介回放的 video 和 audio 元素;
6. 新的文档属性 document.visibilityState

删除：

1. big 、basefont、font等纯表现的元素
2. 对可用性产生负面影响的元素 如 frame frameset

### HTML语义化

1. 更方便开发者阅读 提高开发效率（即使没有CSS的情况下） 例如b 和strong 一个只是单纯的样式强调 一个会重读
2. 搜索引擎爬虫也依赖于HTML标记来确定上下文和各关键字的权重，利于SEO
3. 用正确的标签做正确的事情
4. 使阅读源码的同学更容易理解和分块
5. 对一些有障碍的人士使用仪器阅读

### 前端需要注意哪些 SEO ？

1. TDK合理的设置 方便爬虫爬取收录
2. 语义化标签  爬虫也依赖于HTML标记来确定上下文和各关键字的权重
3. 重要的HTML代码放最前面
4. 嵌套层级不要太深
5. 重要的内容不要用js输出
6. 提高网站的速度
7. 非装饰性的img一定要加上alt



### iframe 有那些缺点？

```
 iframe 元素会创建包含另外一个文档的内联框架（即行内框架）。

 主要缺点有：

 （1） iframe 会阻塞主页面的 onload 事件。window 的 onload 事件需要在所有 iframe 加载完毕后（包含里面的元素）才
      会触发。在 Safari 和 Chrome 里，通过 JavaScript 动态设置 iframe 的 src 可以避免这种阻塞情况。
 （2） 搜索引擎的检索程序无法解读这种页面，不利于网页的 SEO 。
 （3） iframe 和主页面共享连接池，而浏览器对相同域的连接有限制，所以会影响页面的并行加载。
 （4） 浏览器的后退按钮失效。
 （5） 小型的移动设备无法完全显示框架。
```



### Label 的作用是什么？是怎么用的？

```
 label 标签来定义表单控制间的关系，当用户选择该标签时，浏览器会自动将焦点转到和标签相关的表单控件上。

 <label for="Name">Number:</label>
 <input type=“text“ name="Name" id="Name"/>
```



#### 如何实现浏览器内多个标签页之间的通信?

```
 （1）使用 WebSocket，通信的标签页连接同一个服务器，发送消息到服务器后，服务器推送消息给所有连接的客户端。

 （2）使用 SharedWorker （只在 chrome 浏览器实现了），两个页面共享同一个线程，通过向线程发送数据和接收数据来实现标
     签页之间的双向通行。

 （3）可以调用 localStorage、cookies 等本地存储方式，localStorge 另一个浏览上下文里被添加、修改或删除时，它都会触
     发一个 storage 事件，我们通过监听 storage 事件，控制它的值来进行页面信息通信；

 （4）如果我们能够获得对应标签页的引用，通过 postMessage 方法也是可以实现多个标签页通信的。
```







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











## 小程序

### 区别

1. 提出页面的概念，页面有自己的生命周期，onShow onHide onReady
