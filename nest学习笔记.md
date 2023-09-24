## Nest小册学习笔记



### 5种HTTP传输方式

1. url param 如page/:id 

2. query 如page?pageSize=9

3. Form-urlencoded 通过将query字符串放到body里

4. Form-data 适合传输文件

5. json 最常见的 前后端可以通过json.stringfy

   

### IOC

1. 好处 可以控制反转 根据token去自动创建类， 例如Contoller -> Service -> Repository -> DataSource -> Config 依赖关系可以反转
2. Controller 可以被注入
3. Service 可以被注入 也可以注入到其他地方



### 生命周期和全局模块

1. 通过加装饰器@Global()注册全局模块
2. 生命周期先模块内controller provider的onModuleInit 再 module的onModuleInit 接着是 onApplicationBootstrap
3. 销毁的时候 onModuleDestory、beforeApplicationShutdown 、beforeApplicationShutdown



### AOP架构

AOP : 面向切片编程，在我们需要加入一些通用逻辑的时候，为了不侵入业务逻辑，我们则可以通过AOP去进行加入

![img](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f99087120e847eab901738bf8504d21~tplv-k3u1fbpfcp-jj-mark:4536:0:0:0:q75.awebp)

这样就可以优雅的解决加入通用逻辑的问题

##### Nest中的AOP

1. Guard 守卫 例如可以做权限守卫
2. Interceptor 拦截器 加入一些通用的逻辑再Controller方法前后
3. Pipe 用来对参数进行一些校验和转换
4. ExceptionFilter 对抛出的异常进行处理，返回对应的响应

具体的可以看Nest中文文档/Nest通关秘籍 

Nest 就是通过这种 AOP 的架构方式，实现了松耦合、易于维护和扩展的架构。



