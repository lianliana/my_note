# **基于****预渲染****技术的网站SEO优化方案**

1. # **场景及目的**

针对SPA应用提供的一同通用更新通知、静态页面生成、发布的方案，用于实现SEO。

1. # **达到该目的目前市面的技术方案**

目前有现成的库可以在网站打包构建的时候提前做页面预渲染，底层核心是Google的puppeteer库，可以实现服务器环境下的模拟Chrome浏览器访问行为，来保存js执行后生成的页面代码。

基于wiz需要对项目、文章等动态数据做曝光的需求，本方案会支持根据后端的行为进行动态更新静态页面，实现了静态页面的实时性。

1. # **我的技术及创新点**

1. 持续渲染、持续发布，支持根据后端数据动态预渲染并发布
2. 前端代码提供一个工具类，提供针对页面的tdk（title/description/keywords）设定，a标签等。
3. 支持生成sitemap.xml文件，暴露全站需要公开的内容。

1. # **代码结构**

src包括了项目的配置与源代码：

1. config.js 项目的所有配置信息
2. http.js 针对本地调试环境提供的测试接口，访问http://[ip]:[port]/statics0?url=[需要添加进sitemap.xml的内链]即可添加新预渲染任务，新增或更新均可。访问http://[ip]:[port]/statics1即可重新预渲染config.js中预设的加速路径（statics1List）。
3. index.js 启动入口
4. request.js 使用promise封装了request请求
5. sitemap.js 预设的需要用于添加进sitemap.xml的url。不预设也可以，可以在config.js中设置sitemapApi来从后端服务器请求所有需要曝光的urls。这里可以加入'/'，'/project'。
6. statics0renderer.js 针对sitemap的渲染器，新增任务只需要调用s0.render([url])即可。渲染完成后会更新sitemap.xml并发布只cos。
7. statics1renderer.js 针对页面加速的渲染器，新增加速页面需要修改config.js中的statics1List，重新生成加速页面只需要调用s1.rePrerender()
8. template.js 针对sitemap预渲染的结果，由于只需要给爬虫使用（用户访问还需跳转至原始的页面），因此很多内容是可以省略的。使用template.js的目的是简化预渲染结果的体积，从页面中提取出对seo有意义的关键信息，填充进template.js中，生成简化版的html。
9. upload2Cos.js 发布内容到cos
10. redis.js 生产环境通过发布订阅与后端服务器进行通信，作用与http.js相同，只是环境不同。（预渲染服务已经订阅了statics0与statics1频道，服务器需要通过'publish statics0 /project/detail/1399'来通知更新）

statics0存储了针对sitemap的预渲染结果，sitemap.xml与html

statics1存储了针对首页加速的预渲染的结果，针对常见的访问入口进行预渲染优化。

1. # **config.js参数**

1. statics0prefix 针对sitemap的预渲染的url前缀（一般应当为https://xxx.withzz.com/prerender）
2. statics1prefix 针对页面加速的预渲染url前缀（一般应当为https://xxx.withzz.com/prerender）
3. statics1List 需要加速的页面url
4. sitemapApi 若配置，则会在每次服务启动时根据此地址从服务器拉取一个urls的数组（数组中的urls无需包含前缀与query参数，仅需类似‘/project/detail/1234’），来作为需要暴露的urls的依据。sitemapApi的数据（集合a）会与sitemap.js（集合b）中的合并，作为需要预渲染的页面总和，比对本地statics0（集合c）中后发现不存在的页面会依次进行预渲染。（a+b-c是需要被新渲染的内容）
5. sitemapPrefix sitemap中的网址前缀，比如预渲染的url是http://dev.withzz.com/project/detail/1377，结果需要通过http://dev.withzz.com/p/project/detail/1377暴露给搜索引擎，那么sitemapPrefix设置为http://dev.withzz.com/p。
6. sitemapPublicPath sitemap需要被发布在cos上的路径
7. statics0PublicDir statics0中的html发布在cos上的文件夹路径（可以给'/p'，代表发布到比如'https://xxx.withzz.com/p'）
8. statics1PublicDir statics1中的html发布在cos上的文件夹路径（应当给''，表示发布到'https://xxx.withzz.com'）
9. cos 腾讯云cos sdk的参数
10. redis 与后端服务器通信的redis服务器参数
11. http 本地http调试的参数
12. statics0ParallelLimit statics0渲染并行工作数
13. statics1ParallelLimit statics1渲染并行工作数

1. # **流水线改造**

1. 每次前端流水线重新构建时，必须重新使用statics1渲染器进行重新预渲染，以确保用户访问的加速页面或得到了最新的依赖文件。
2. 由于statics1渲染的结果通常是发布在cos根目录上的，因此当重新构建时，需要注意statics1prefix的配置。如果cos根目录的加速页面html不会被清理，则应避免statics1prefix是根目录，否则每次重新渲染都会获取到老旧的依赖文件。

1. # **服务启动**

node src/index.js [dev/ty/master] [http/redis]

1. # **流程图**

整体运行流程

​                 ![img](https://docimg7.docs.qq.com/image/4HIlwFM5yxJZ1oPcn5bmmw.png?w=1280&h=728.8888888888888)        	statics0渲染范围确定方法

​                 ![img](https://docimg4.docs.qq.com/image/Guoj5JE4SuFwfGDMK_Vo3g.png?w=1280&h=898.4491440080565)        	statics0 render流程

​                 ![img](https://docimg9.docs.qq.com/image/IVeY4ibPdtqdq_iTtaEv7Q.png?w=1280&h=314.40699126092386)        

statics1 render流程

​                 ![img](https://docimg5.docs.qq.com/image/MIIkonRDW10ZtWnD-AdDFw.png?w=1280&h=454.19354838709677)        