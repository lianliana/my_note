**异步 非阻塞IO 单线程 事件驱动** Node

##### 特点/优势

1. IO密集型
2. CPU密集型也还可以
3. 分布式应用





### node模块化

- 自定义模块
- 第三方模块
- 内置模块

自定义模块中module.exports为{} 空对象

导入模块的时候会执行模块内的代码，并且缓存

__使用require导入模块永远以module.exports指向的对象为准__    切记__对象__！！

#### module

- exports
- filename
- paths

#### export

__默认情况下__,exports和module.exports指向同一个对象，后面的操作就是指针操作

#### CommonJS规定

- 每个模块内部，module变量代表当前模块
- moudule变量是一个对象，它的exports属性(即module.exports)是对外的接口
- 加载某个模块，其实是加载该模块的module.exports属性，requre()方法用于加载模块



#### 加载包

指定路径是目录：先找Package.json 然后 index.js 最后

指定的是第三方模块 会递归查找每一级目录下的node_modules

### 中间件middleware

全局中间件是使用app.use进行注册，并且中间件中要调用next来转交给其他的中间件或路由

不使用app.use的中间件叫做局部中间件，只会影响局部的中间件/路由

#### 注意事项

- 一定要在路由之前注册中间件
- 中间件要记得调用next()
- 调用完next后不要写代码(为了逻辑清晰)
- 可以连续调用多个中间件
- 连续调用多个中间件时，多个中间件之间，共享req和res对象

#### 中间件分类

- 应用级别的中间件: 绑定到app实例上的中间件 如app.use() app.get() app.post()
- 路由级别的中间件: 绑定到Express.router实例上的
- 错误级别的中间件: 专门为了捕获错误 **必须在路由之后**
- Express内置的中间件
- 第三方的中间件

### JWT

- json web token 
- 三部分组成  1.header 2.payload 3.signature
- npm install jsonwebtoken(生成jwt) express-jwt(解析)
- npm i jsonwebtoken@8.5.1 npm i express-jwt@5.3.3

### 参数校验

- expressJoi 参数验证

流程

1. npm install @hapi/joi@17/.1.0  //定义验证规则的包
2. npm i @escook/express-joi  //验证数据的中间件
3. 











### Node深入浅出



#### 异步IO

1. 事件循环
2. 观察者
3. 请求对象

具体流程是 单线程执行 遇到异步IO就创建请求对象丢入线程池中等待执行 线程池中调用完后丢入IO观察者进行事件循环



#### 异步编程

##### 解决方案

- 事件发布和订阅 EventEmitter
- Promise/Deferred模式 Promise等
- 流程控制库 例如express等中的next流程控制串行





#### 内存

##### 内存泄露

原因：

- 缓存 例如用memo={}去缓存数据  这时候一旦访问量大 可能会爆内存 所以最好使用缓存算法例如LRU或使用Redis等成熟的方案
- 队列消费不及时 例如IO操作 写的速度比较慢 导致堆积了很多缓存
- 作用域未释放 例如闭包

###### 排查内存泄漏

node-heapdump、 node-memwatch









## 面试题

#### require 模块引入的查找方式？

```js
当 Node 遇到 require(X) 时，按下面的顺序处理。

（1）如果 X 是内置模块（比如 require('http')）
　　a. 返回该模块。
　　b. 不再继续执行。

（2）如果 X 以 "./" 或者 "/" 或者 "../" 开头
　　a. 根据 X 所在的父模块，确定 X 的绝对路径。
　　b. 将 X 当成文件，依次查找下面文件，只要其中有一个存在，就返回该文件，不再继续执行。
    X
    X.js
    X.json
    X.node

　　c. 将 X 当成目录，依次查找下面文件，只要其中有一个存在，就返回该文件，不再继续执行。
    X/package.json（main字段）
    X/index.js
    X/index.json
    X/index.node

（3）如果 X 不带路径
　　a. 根据 X 所在的父模块，确定 X 可能的安装目录。
　　b. 依次在每个目录中，将 X 当成文件名或目录名加载。

（4）抛出 "not found"
```



### Node事件循环

```js
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
   
   

```

[事件循环资料](https://juejin.cn/post/6844903999506923528#comment)

[思否事件循环](https://segmentfault.com/a/1190000040364902)

## 事件循环详解



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/11/18/16e7d83cb93ba59e~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)

这个图是整个 Node.js 的运行原理，从左到右，从上到下，Node.js 被分为了四层，分别是 `应用层`、`V8引擎层`、`Node API层` 和 `LIBUV层`。



> - 应用层：   即 JavaScript 交互层，常见的就是 Node.js 的模块，比如 http，fs
> - V8引擎层：  即利用 V8 引擎来解析JavaScript 语法，进而和下层 API 交互
> - NodeAPI层：  为上层模块提供系统调用，一般是由 C 语言来实现，和操作系统进行交互 。
> - LIBUV层： 是跨平台的底层封装，实现了 事件循环、文件操作等，是 Node.js 实现异步的核心





1. **`timers` 阶段**: 这个阶段执行 `setTimeout(callback)` 和 `setInterval(callback)` 预定的 callback;

2. **`I/O callbacks` 阶段**: 此阶段执行某些系统操作的回调，例如TCP错误的类型。 例如，如果TCP套接字在尝试连接时收到 ECONNREFUSED，则某些* nix系统希望等待报告错误。 这将操作将等待在==I/O回调阶段==执行;

3. **`idle, prepare` 阶段**: 仅node内部使用;

4. **`poll` 阶段**: 获取新的I/O事件, 例如操作读取文件等等，适当的条件下node将阻塞在这里;

5. **`check` 阶段**: 执行 `setImmediate()` 设定的callbacks;

6. **`close callbacks` 阶段**: 比如 `socket.on(‘close’, callback)` 的callback会在这个阶段执行;

