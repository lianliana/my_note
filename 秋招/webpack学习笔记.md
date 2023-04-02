#### webpack解决了什么问题

- 解决了作用域的问题
- 解决了代码拆分的问题 commonjs es6模块化
- 如何让浏览器支持模块

面试回答：

- 将ES6以后的语法转换成ES5 less sass 转换成css
- 文件优化 压缩代码
- 公共代码的提取
- 代码变更后能自动构建刷新浏览器

#### wepack5  资源模块 asset module 用来加载js以外的资源

- aseet/resource  发送一个单独的文件并导出url
- asset/inline 导出一个资源的data URL
- asset/source 导出资源的源代码
- asset 到处一个dataURL 和发送一个单独的文件之间自动进行选择

```js
 module:{
        rules:[
            {
                test: /\.png$/,
                type:'asset/resource' ,
                generator:{
                    filename: 'images/test.png'
                }
            },
            {
                test: /\.svg$/,
                type:'asset/inline'
            },
            {
                test: /\.txt$/,
                type:'asset'
            },
            {
                test: /\.jgp$/,
                type:'asset',
                parser:{
                    dataUrlCondition:{
                        maxSize:4*1024*1024
                    }
                }
            }
         }
```

加载fonts字体

```js	
{
    test:/\.(woff|woff2|eot|ttf|otf)$/,
    type:'asset/resource'
}
```



#### 常用的Plugin

###### HtmlWebpackPlugin  

- 自动生成index.html去引入bundle.js

```js
plugins:[
    //基于index.html来生成新的html
        new HtmlWebpackPlugin({
            template: './index.html',
            filename: 'app.html',
            inject: 'body'
        })
    ]
```



###### MiniCssExtractPlugin  

- 通过外部引入打包好的css  命令：npm install mini-css-extract-plugin -d

```js
plugins:[
        new MiniCssExtractPlugin({
            filename:'styles/[contenthash].css'
        })
    	]
module:{
        rules:[
            {
                test: /\.(css|less)$/,
                use: [MiniCssExtractPlugin.loader,'css-loader','less-loader']
            }
        ]
    }
```



###### CssMinimizerPlugin  

- 压缩css文件 提高生产环境下执行效率  命令：npm install css-minimizer-webpack-plugin -d

```js
mode:'production',
optimization:{
        minimizer:[
            new CssMinimizerPlugin()
        ]
    }
```



###### TerserPlugin

- 压缩js文件 提高生产环境下执行效率 命令：npm install terser-webpack-plugin -d

```js
optimization:{
            minimizer:[
                new CssMinimizerPlugin(),
                new TerserPlugin()
            ],
}
```



###### purgecss-webpack-plugin

```js
const config = {
  plugins:[ // 配置插件
    // ...
    new PurgecssPlugin({
      paths: glob.sync(`${PATHS.src}/**/*`, {nodir: true})
    }),
  ]
}

```





```js
...
// 费时分析
const SpeedMeasurePlugin = require("speed-measure-webpack-plugin");
const smp = new SpeedMeasurePlugin();
...

const config = {...}

module.exports = (env, argv) => {
  // 这里可以通过不同的模式修改 config 配置


  return smp.wrap(config);
}
```



###### webpack-bundle-analyzer 巨好用

可视化打包产物依赖图

tip:安装命令为 npm install --save-dev webpack-bundle-analyzer

```js
module.exports = {
    plugins: [
        // ...others
        new BundleAnalyzerPlugin()
    ]
}

```





###### CommonsChunkPlugin

```js
module.exports = {
    plugins: [
        // ...others
        new webpack.optimize.CommonsChunkPlugin({
          name: 'vendor',
        }),
    ]
}
```



###### clean-webpack-plugin

```js
// 引入插件
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
module.exports = {
  // ...
  plugins:[ // 配置插件
    new HtmlWebpackPlugin({
      template: './src/index.html'
    }),
    new CleanWebpackPlugin() // 引入插件
  ]
}
```









#### 常见的Loader  

**让webpack 可以去处理其他格式的文件并且将他们转化为有效的模块**

###### babal-loader 

- 将ES6转换成ES5 要安装babel-loader 应用bael解析ES6的桥梁 

  @babel/core babel核心模块

   babel/preset-env  babel预设，一组babel插件的集合

```js
module:{
        rules:[
		{
                test:/\.js$/,
                exclude:/node_modules/,
                use:{
                    loader: 'babel-loader',
                    options:{
                        presets:['@babel/preset-env'],
                        plugins:[
                            [
                                '@babel/plugin-transform-runtime'
                            ]
                        ]
                    }
                }
            }
            ]
}
```



###### css-loader

-  webpack自动打包转换css文件



###### style-loader 

- webpack打包自动生成style标签引入



###### less-loader 

- webpack打包自动转换less文件  tip:安装命令为 npm install less-loader less -d

```js
module:{
        rules:[
            {
                test: /\.(css|less)$/,
                use: ['style-loader','css-loader','less-loader'] //栈，右右边往左边
            }
        ]
    }
```



###### postcss-loader

```js
const config = {
  module: { 
    rules: [
      {
        test: /\.css$/, //匹配所有的 css 文件
        use: ['style-loader','css-loader', 'postcss-loader']
      }
    ]
  },
}

// postcss.config.js
module.exports = {
  plugins: [require('postcss-preset-env')]
}

//创建 postcss-preset-env 配置文件 
//.browserslistrc
# 换行相当于 and
last 2 versions # 回退两个浏览器版本
> 0.5% # 全球超过0.5%人使用的浏览器，可以通过 caniuse.com 查看不同浏览器不同版本占有率
IE 10 # 兼容IE 10

```



常用的处理图片文件的 Loader 包含：

| Loader      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| file-loader | 解决图片引入问题，并将图片 copy 到指定目录，默认为 dist      |
| url-loader  | 解依赖 file-loader，当图片小于 limit 值的时候，会将图片转为 base64 编码，大于 limit 值的时候依然是使用 file-loader 进行拷贝 |
| img-loader  | 压缩图片                                                     |



```JS
const config = {
  //...
  module: { 
    rules: [
      {
         // ...
      }, 
      {
        test: /\.(jpe?g|png|gif)$/i, // 匹配图片文件
        use:[
          'file-loader' // 使用 file-loader
        ]
      }
    ]
  },
  // ...
}

```



url-loader

```js
const config = {
  //...
  module: { 
    rules: [
      {
         // ...
      }, 
      {
        test: /\.(jpe?g|png|gif)$/i,
        use:[
          {
            loader: 'url-loader',
            options: {
              name: '[name][hash:8].[ext]',
              // 文件小于 50k 会转换为 base64，大于则拷贝文件
              limit: 50 * 1024
            }
          }
        ]
      },
    ]
  },
  // ...
}
```







###### imports-loader



###### csv-loader xml-loader

```js	
module:{
        rules:[
            {
                test: /\.(csv|tsv)$/,
                use:'csv-loader'
            },
            {
                test:/\.xml$/,
                use:'xml-loader'
            },
        ]
    }
```

######  自定义JSON模块parser

-  toml yaml json5

```js
module:{
        rules:[
            {
                test:/\.toml$/,
                type: 'json',
                parser:{
                    parse: toml.parse
                }
            },
            {
                test:/\.yaml$/,
                type: 'json',
                parser:{
                    parse: yaml.parse
                }
            },
            {
                test:/\.json5l$/,
                type: 'json',
                parser:{
                    parse: json5.parse
                }
            }
        ]
    }

```



###### cache-loader

```js
const config = {
 module: { 
    // ...
    rules: [
      {
        test: /\.(s[ac]|c)ss$/i, //匹配所有的 sass/scss/css 文件
        use: [
          // 'style-loader',
          MiniCssExtractPlugin.loader,
          'cache-loader', // 获取前面 loader 转换的结果
          'css-loader',
          'postcss-loader',
          'sass-loader', 
        ]
      }, 
      // ...
    ]
  }
}
```



###### HotModuleReplacementPlugin





##### awesom-typescript-loader

可以让Webpack使用TypeScript的标准配置文件`tsconfig.json`编译TypeScript代码



#### Resolve配置

1. alias:创建Import或require模块的别名，使得模块导入变得简单
2. extensions:当不带拓展名的模块导入时 会按照extensions的数组顺序依次进行查询
3. 





#### 常用的代码分离方法

1. 使用entry 配置手动地分离代码

```js
module.exports={
    entry:{
        index:{
            import: './src/index.js',
            depenOn: 'shared'
        },
        another:{
            import: './src/another-module.js',
            dependOn: 'shared'
        },
        shared: 'loadsh'
    },
    output:{
    filename:'[name].bundle.js'
		   }
}
```



2. 使用Entry dependencied 或者 SplitChunksPlugin 去重和分离代码

```js
module.exports={
    entry:{
        index: './src/index.js',
        another: './src/another-module.js'
    },
    optimization:{
        splitChunks:{
            chunks: 'all'
        }
    },
    output:{
    filename:'[name].bundle.js'
	}
}
```



3. 通过模块的内联函数调用来分离代码  动态导入

```js
实现按需引入
const button =document.createElement('button')
button.textContent="点击执行加法"
button.addEventListener('click',()=>{
    import(/* webpackChunkName: 'math' */'./math.js').then(({add})=>{
        console.log(add(4,5))
    })
})
document.body.appendChild(button)
```





##### 预获取和预加载

```js
//prefetch是在网络空闲的时候先去获取资源
const button =document.createElement('button')
button.textContent="点击执行加法"
button.addEventListener('click',()=>{
    import(/* webpackChunkName: 'math' webpackPrefetch: true*/'./math.js').then(({add})=>{
        console.log(add(4,5))
    })
})
document.body.appendChild(button)
```

##### 缓存相关

自己写的代码更新缓存

```js
output:{
    	//更换名字来更新缓存
        filename:'[name].[contenthash].js',
        path:path.resolve(__dirname,'./dist'),
        clean:true, 
        assetModuleFilename: 'images/[contenthash].[ext]'
    },
```

第三方库走缓存

```js
optimization:{
        minimizer:[
            new CssMinimizerPlugin()
        ],
        splitChunks:{
            cacheGroups:{
                //打包时不修改第三方库，进行缓存
                vendor:{
                    test: /[\\/]node_modules[\\/]/,
                    name:'vendors',
                    chunks: 'all'
                }
            }
        }
    }
```



## Webpack 高级篇

[source map掘金资料](https://juejin.cn/post/7023242274876162084#heading-17)

#### source map

- 默认是开启 eval模式的source map   除非devtool:false
- devtool: source-map   生成一个SourceMap文件
- devtool: hidden-source-map  和source-map 一样，但不会再bundle末尾追加注释
- devtool: inline-source-map 不会单独把sourcemap文件生成出来 但是有注释source url
- devtool: eval-source-map 生成一个dataurl形式的sourcemap 每个module都会通过eval()来执行
- devtool: cheap-source-map 简化版的souce-map 只map行信息的source-map
- devtool: cheap-module-source-map  引入babel之后也不会影响source-map的结果

**生产环境我们一般不开启source-map，减少项目体积以及防止反编译**



#### devServer

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
const path = require("path");
module.exports={
    mode: 'development',
    entry: './app.js',
    devServer:{
        static: path.resolve(__dirname,'./dist'), //资源路径
        compress: true, //是否开始压缩
        port:8081, //devServer开启的端口号
        headers:{
            'X-Access-Token': 'abc123'
        },
        proxy:{
            '/api': 'http://localhost:9000',   
        },

        https:true,//使用https
        http2:true,//使用http2
        historyApiFallback: true //任何路由都可以配置跳转 ，防止404
    },

    plugins :[
        new HtmlWebpackPlugin()
    ]
}
```



#### 模块热替换与热加载

模块热替换(HMR - hot module replacement)功能会在应用程序运行过程中， 替换、添加或删除 模块，而无需重新加载整个页面



- 启用 webpack 的 热模块替换 特性，需要配置devServer.hot参数

```js
module.exports = {
//...
devServer: {
	hot: true,
	},
};
```

- 热加载

```js
module.exports = {
//...
    devServer: {
    	liveReload: false, //默认为true，即开启热更新功能。
    },
};

```





#### 模块解析

通过内置的enhanced-resolve，webpack 能解析三种文件路径：

- 相对路径
- 绝对路径
- 模块路径

```js
// webpack.config.js
const path = require('path');
module.exports = {
    //...
    resolve: {
        alias: {
            "@utils": path.resolve(__dirname, 'src/utils/')  //取别名 可直接@utils访问到src目录下的utils
        },
        extensions: ['.js', '.json', '.wasm'],//按照数组的吮吸取解析这些后缀名
    },
};
```



```js
module.exports={
    //...
    externals:{
        jquery: 'jQuery' //配日志外部拓展模块 JQuery是cdn里打入到window的变量名
    }
}
```



#### PostCSS和CSS模块

- PostCSS

  ```js
  //webpack.config.js
  module.exports = {
      module: {
      rules: [
          {
              test: /\.css$/,
              exclude: /node_modules/,
              use: [
                      {
                          loader: 'style-loader',
                      },
                      {
                          loader: 'css-loader',
                          options: {
                              importLoaders: 1,
                              modules: true //开启css模块 React和Vue中有使用 这样css类名是唯一的
                          }
                      },
                      {
                          loader: 'postcss-loader'
                      }
              ]
          }
      ]
      }
  }
  ```

  ```js
  //postcsss.config.js
  module.exports = {
      plugins: [
          require('autoprefixer'),
          require('postcss-nested')
      ]
   }
  ```

  ```js
  //package.json
  "browserslist":[
      '> 1%',
      "last 2 versions"
  ]
  ```

  

#### TypeScript

**在webpack工程化环境中集成TS**  

命令：npm install --save-dev typescript ts-loader  

npx tsc --init ts的配置文件

```js
{
    "compilerOptions": {
        "outDir": "./dist/",
        "noImplicitAny": true,
        "sourceMap": true,
        "module": "es6",
        "target": "es5",
        "jsx": "react",
        "allowJs": true,
        "moduleResolution": "node"
    }
}
```



```js
//webpack.config.js
const path = require('path');
    module.exports = {
        entry: './src/index.ts',
        devtool: 'inline-source-map',
        module: {
            rules: [
                {
                    test: /\.(ts|tsx)$/,
                    use: 'ts-loader',
                    exclude: /node_modules/,
                },
            ],
   		 },
    resolve: {
        extensions: [ '.tsx', '.ts', '.js' ],
        },
        output: {
            filename: 'bundle.js',
            path: path.resolve(__dirname, 'dist'),
    },
};
```





#### Tree shaking

通常用于描述移除 JavaScript 上下文中的未引用代码 (dead-code)。它依赖于 ES2015 模块语法的 静态结构 特性

```js
//webpack5默认开启树摇
const path = require('path');
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist'),
    },
    mode: 'production',
};
```

##### sideEffects

它有三个可能的值： 

- true 如果不指定其他值的话。这意味着所有的文件都有副作用，也就是没有一个文件 可以 tree-shaking。 
- false 告诉 Webpack 没有文件有副作用，所有文件都可以 tree-shaking。 
- 数组[…]





#### 渐进式网络应用程序 PWA

是一种可以提供类似于 native app(原生应用程序) 体验的 web app(网络应用程序)。PWA 可以用来做很多 事。其中最重要的是，在离线(offline)时应用程序能够继续运行功能。这是通过使用 名为 Service Workers 的 web 技术来实现的。



##### workbox-webpack-plugin

命令：npm install workbox-webpack-plugin --save-dev

添加 workbox-webpack-plugin 插件，然后调整 webpack.config.js 文件：

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WorkboxPlugin = require('workbox-webpack-plugin');
module.exports = {
    entry: {
    	app: './src/index.js',
    },
    plugins: [
        new HtmlWebpackPlugin(),
        new WorkboxPlugin.GenerateSW({
            // 这些选项帮助快速启用 ServiceWorkers
            // 不允许遗留任何“旧的” ServiceWorkers
            clientsClaim: true,
            skipWaiting: true,
        }),
    ],
    output: {
        filename: '[name].bundle.js',
        path: path.resolve(__dirname, 'dist'),
        clean: true,
    },
}

```

**注册Service Worker**

```js
//...
if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
    	navigator.serviceWorker.register('/serviceworker.js')
            .then(registration=> {
    	console.log('SW registered: ', registration);
    }).catch(registrationError => {
    	console.log('SW registration failed: ', registrationError);
    });
    });
}
```

#### Shimming

##### 细粒度Shimming 

一些遗留模块依赖的 this 指向的是 window 对象。当模块运行在 CommonJS 上下文中，这将会变成一个问题，也就是说此时的 this 指向的是 module.exports 。在这种情况下，你可以通过使用 imports-loader 覆 盖 this 指向：

```js
const webpack = require('webpack')
const HtmlWebpackPlugin = require('html-webpack-plugin')
module.exports = {
    mode: 'production',
    entry: './src/index.js',
    
    module: {
        rules: [
            {
                test: require.resolve('./src/index.js'),
                use: 'imports-loader?wrapper=window',
            },
    	]
    },
    plugins: [
    	new webpack.ProvidePlugin({
    		_: 'lodash'
    	}),
    	new HtmlWebpackPlugin()
    ]
}
```





### runtime运行时

https://www.jianshu.com/p/714ce38b9fdc

设置runtimeChunk是将包含`chunks 映射关系`的 list单独从 app.js里提取出来，因为每一个 chunk 的 id 基本都是基于内容 hash 出来的，所以每次改动都会影响它，如果不将它提取出来的话，等于app.js每次都会改变。缓存就失效了。设置runtimeChunk之后，webpack就会生成一个个runtime~xxx.js的文件。
 然后每次更改所谓的运行时代码文件时，打包构建时app.js的hash值是不会改变的。如果每次项目更新都会更改app.js的hash值，那么用户端浏览器每次都需要重新加载变化的app.js，如果项目大切优化分包没做好的话会导致第一次加载很耗时，导致用户体验变差。现在设置了runtimeChunk，就解决了这样的问题。所以`这样做的目的是避免文件的频繁变更导致浏览器缓存失效，所以其是更好的利用缓存。提升用户体验。`
#### prefetch 与 preload

上面我们使用异步加载的方式引入图片的描述，但是如果需要异步加载的文件比较大时，在点击的时候去加载也会影响到我们的体验，这个时候我们就可以考虑使用 prefetch 来进行预拉取

##### prefetch

> - **prefetch** (预获取)：浏览器空闲的时候进行资源的拉取

改造一下上面的代码

```js
// 按需加载
img.addEventListener('click', () => {
  import( /* webpackPrefetch: true */ './desc').then(({ default: element }) => {
    console.log(element)
    document.body.appendChild(element)
  })
})
```

##### preload

> - **preload** (预加载)：提前加载后面会用到的关键资源
> - ⚠️ 因为会提前拉取资源，如果不是特殊需要，谨慎使用

官网示例：

```js
import(/* webpackPreload: true */ 'ChartingLibrary');
```













## 面试题

## Loader和Plugin的区别

`Loader` 本质就是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果。 因为 Webpack 只认识 JavaScript，所以 Loader 就成了翻译官，对其他类型的资源进行转译的预处理工作。

`Plugin` 就是插件，基于事件流框架 `Tapable`，插件可以扩展 Webpack 的功能，在 Webpack 运行的生命周期中会广播出许多事件，Plugin 可以监听这些事件，在合适的时机通过 Webpack 提供的 API 改变输出结果。

`Loader` 在 module.rules 中配置，作为模块的解析规则，类型为数组。每一项都是一个 Object，内部包含了 test(类型文件)、loader、options (参数)等属性。

`Plugin` 在 plugins 中单独配置，类型为数组，每一项是一个 Plugin 的实例，参数都通过构造函数传入。



## 使用webpack开发时，你用过哪些可以提高效率的插件？

(这道题还蛮注重实际，用户的体验还是要从小抓起的)

- `webpack-dashboard`：可以更友好的展示相关打包信息。
- `webpack-merge`：提取公共配置，减少重复配置代码
- `speed-measure-webpack-plugin`：简称 SMP，分析出 Webpack 打包过程中 Loader 和 Plugin 的耗时，有助于找到构建过程中的性能瓶颈。
- `size-plugin`：监控资源体积变化，尽早发现问题
- `HotModuleReplacementPlugin`：模块热替换





## 说一下 Webpack 的热更新原理吧

`Webpack` 的热更新又称热替换（`Hot Module Replacement`），缩写为 `HMR`。 这个机制可以做到不用刷新浏览器而将新变更的模块替换掉旧的模块。

HMR的核心就是客户端从服务端拉去更新后的文件，准确的说是 chunk diff (chunk 需要更新的部分)，实际上 WDS 与浏览器之间维护了一个 `Websocket`，当本地资源发生变化时，WDS 会向浏览器推送更新，并带上构建时的 hash，让客户端与上一次资源进行对比。客户端对比出差异后会向 WDS 发起 `Ajax` 请求来获取更改内容(文件列表、hash)，这样客户端就可以再借助这些信息继续向 WDS 发起 `jsonp` 请求获取该chunk的增量更新。

后续的部分(拿到增量更新之后如何处理？哪些状态该保留？哪些又需要更新？)由 `HotModulePlugin` 来完成，提供了相关 API 以供开发者针对自身场景进行处理，像`react-hot-loader` 和 `vue-loader` 都是借助这些 API 实现 HMR。

细节请参考[Webpack HMR 原理解析](https://link.juejin.cn?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F30669007)





## 文件指纹是什么？怎么用？

文件指纹是打包后输出的文件名的后缀。

- `Hash`：和整个项目的构建相关，只要项目文件有修改，整个项目构建的 hash 值就会更改
- `Chunkhash`：和 Webpack 打包的 chunk 有关，不同的 entry 会生出不同的 chunkhash
- `Contenthash`：根据文件内容来定义 hash，文件内容不变，则 contenthash 不变





## 如何优化 Webpack 的构建速度？

(这个问题就像能不能说一说**「从URL输入到页面显示发生了什么」**一样）

(我只想说：您希望我讲多长时间呢？)

(面试官：。。。)

- 使用`高版本`的 Webpack 和 Node.js
- `多进程/多实例构建`：HappyPack(不维护了)、thread-loader
- `压缩代码`
  - 多进程并行压缩
    - webpack-paralle-uglify-plugin
    - uglifyjs-webpack-plugin 开启 parallel 参数 (不支持ES6)
    - terser-webpack-plugin 开启 parallel 参数
  - 通过 mini-css-extract-plugin 提取 Chunk 中的 CSS 代码到单独文件，通过 css-loader 的 minimize 选项开启 cssnano 压缩 CSS。
- `图片压缩`
  - 使用基于 Node 库的 imagemin (很多定制选项、可以处理多种图片格式)
  - 配置 image-webpack-loader
- `缩小打包作用域`：
  - exclude/include (确定 loader 规则范围)
  - resolve.modules 指明第三方模块的绝对路径 (减少不必要的查找)
  - resolve.mainFields 只采用 main 字段作为入口文件描述字段 (减少搜索步骤，需要考虑到所有运行时依赖的第三方模块的入口文件描述字段)
  - resolve.extensions 尽可能减少后缀尝试的可能性
  - noParse 对完全不需要解析的库进行忽略 (不去解析但仍会打包到 bundle 中，注意被忽略掉的文件里不应该包含 import、require、define 等模块化语句)
  - IgnorePlugin (完全排除模块)
  - 合理使用alias
- `提取页面公共资源`：
  - 基础包分离：
    - 使用 html-webpack-externals-plugin，将基础包通过 CDN 引入，不打入 bundle 中
    - 使用 SplitChunksPlugin 进行(公共脚本、基础包、页面公共文件)分离(Webpack4内置) ，替代了 CommonsChunkPlugin 插件
- `DLL`：
  - 使用 DllPlugin 进行分包，使用 DllReferencePlugin(索引链接) 对 manifest.json 引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译浪费时间。
  - HashedModuleIdsPlugin 可以解决模块数字id问题
- `充分利用缓存提升二次构建速度`：
  - babel-loader 开启缓存
  - terser-webpack-plugin 开启缓存
  - 使用 cache-loader 或者 hard-source-webpack-plugin
- `Tree shaking`
  - 打包过程中检测工程中没有引用过的模块并进行标记，在资源压缩时将它们从最终的bundle中去掉(只能对ES6 Modlue生效) 开发中尽可能使用ES6 Module的模块，提高tree shaking效率
  - 禁用 babel-loader 的模块依赖解析，否则 Webpack 接收到的就都是转换过的 CommonJS 形式的模块，无法进行 tree-shaking
  - 使用 PurifyCSS(不在维护) 或者 uncss 去除无用 CSS 代码
    - purgecss-webpack-plugin 和 mini-css-extract-plugin配合使用(建议)
- `Scope hoisting`
  - 构建后的代码会存在大量闭包，造成体积增大，运行代码时创建的函数作用域变多，内存开销变大。Scope hoisting 将所有模块的代码按照引用顺序放在一个函数作用域里，然后适当的重命名一些变量以防止变量名冲突
  - 必须是ES6的语法，因为有很多第三方库仍采用 CommonJS 语法，为了充分发挥 Scope hoisting 的作用，需要配置 mainFields 对第三方模块优先采用 jsnext:main 中指向的ES6模块化语法
- `动态Polyfill`
  - 建议采用 polyfill-service 只给用户返回需要的polyfill，社区维护。 (部分国内奇葩浏览器UA可能无法识别，但可以降级返回所需全部polyfill)





### Webpack和Vite的区别

webpack会先打包，然后启动开发服务器，请求服务器时直接给予打包结果。 而vite是直接启动开发服务器，请求哪个模块再对该模块进行实时编译。 由于现代浏览器本身就支持ES Module，会自动向依赖的Module发出请求。vite充分利用这一点，将开发环境下的模块文件，就作为浏览器要执行的文件，而不是像webpack那样进行打包合并。 由于vite在启动的时候不需要打包，也就意味着不需要分析模块的依赖、不需要编译，因此启动速度非常快。当浏览器请求某个模块时，再根据需要对模块内容进行编译。这种按需动态编译的方式，极大的缩减了编译时间，项目越复杂、模块越多，vite的优势越明显。 在HMR方面，当改动了一个模块后，仅需让浏览器重新请求该模块即可，不像webpack那样需要把该模块的相关依赖模块全部编译一次，效率更高。 当需要打包到生产环境时，vite使用传统的rollup进行打包，因此，vite的主要优势在开发阶段。另外，由于vite利用的是ES Module，因此在代码中不可以使用CommonJS













## 提升构建性能

