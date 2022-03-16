

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
- 
