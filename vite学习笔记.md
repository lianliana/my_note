
## 双引擎架构


## 插件系统

**特点**
- 拓展rollup插件体系
- vite独特的钩子

钩子

bulid

![img](assets/58ce9fa2b0f14dd1bc50a9c849157e43_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0-9659323.awebp)

- resolveId
- load
- transform

output

![img](assets/5dc4935d712d451fb6978fad46dd7b74_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.awebp)

- 


#### 例子
alias插件
- 通过resolveId去进行匹配字符串然后替换
- 接着通过this.resolve()去调用其他插件的resolveId(分工)
- 最后返回第一个resolveId的返回值
- 如果最后不是一个绝对路径 warning提示 因为相对路径的话可能会存在多次导入的问题