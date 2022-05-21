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
