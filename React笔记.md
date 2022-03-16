

### React入门

#### React是什么？

用于构建用户界面的Javascript库——用于将数据渲染成HTML视图的开源Javascript库

#### React的特点

- 声明式编码
- 组件化编码
- 高效(优秀的Diffing算法)

##### 关于虚拟DOM

- 本质是Object类型的对象
- 虚拟DOM属性较少，因为虚拟DOM是React内部在用
- 虚拟DOM最终会被React转换成真实DOM

###### jsx语法规则

- 定义虚拟DOM,不要写引号
- 标签中混入**JS表达式**要用{}     JS表达式：会产生一个值
- 样式的类名指定不要用class 要用 className=""
- 内联样式,要用style={{key:value}}的形式去写
- 虚拟DOM只有一个根标签
- 标签必须闭合  如`<input type="text" />`
- 标签首字母若为小写则转换成同名html标签,大写转换成对应的组件
- Jsx的babel转换的时候会开启严格模式

```react
<script type="text/babel">
const data =["1","2","3"];
const VDOM = (
                <ul>
                    { //{}里面只能放表达式
                        //如果渲染的是一个列表，每一行必须要有一个key
                        data.map((item,index)=>{
                            return <li key = {index}>{item}</li>
                        })
                    }
                </ul>    
        )
        // 2.渲染，如果有多个渲染同一个容器，后面的会将前面的覆盖掉
        ReactDOM.render(VDOM,document.getElementById("test"));
</script>
```



### React面向组件化编程

#### 函数式组件

```react
function Mycomponent(){
    //此处的this指向为undefined 因为此处使用babel会开启严格模式
    return <h1>我是用函数式组件创建的组件（适用于【简单组件】的创建）</h1>
}
ReactDOM.render(<Mycomponent/>,document.getElementById('test'))
```

#### 类组件(有三大属性)

``` react
class Mycomponent extends React.Component{
    render(){
         //render放在Mycomponent的原型对象上 即 Mycomponent.prototype
    	//render中的this指向Mycomponent的实例对象上
        return <h2>我是用类式组件创建的组件（适用于【复杂组件】的创建）</h2>
    }
}
ReactDOM.render(<Mycomponent/>,document.getElementById('test'))
```



### 组件实例的三大属性

#### 三大属性

##### State

```react
class Weather extends React.Component{
      constructor(props){
          //constructor调用1次，this指向实例自身
          super(props);
          this.state={ishot:false};
          this.changeWeather=this.changeWeather.bind(this);
      }
      render(){
          //render调用1+n次，this指向实例自身
          return <h1 onClick={this.changeWeather}>今天天气{this.state.ishot?'炎热':'凉爽'}</h1>;}
      changeWeather(){
          //this指向调用它的对象，只有当实例调用它的时候this才指向Weather的实例
          //render用n次
          const isHot=this.state.ishot;
          //不能直接修改state的值，需要用内部的API setState
          this.setState({ishot:!isHot});
       
  }
      }   
      ReactDOM.render(<Weather/>,document.getElementById('test')); 
```

```react
//上面代码简化版
class Weather extends React.Component{
    //this指向为类实例
    state={isHot:false,wind:"凉爽"}
    render(){
        return <h1 onClick={this.changeWeather}>今天天气{this.state.isHot?'炎热':'凉爽'}</h1>
    }
    //使用箭头函数来保证this指向不丢失
    changeWeather=()=>{
        const isHot=this.state.isHot
        this.setState({isHot:!isHot})
    }
}
ReactDOM.render(<Weather/>,document.getElementById('test'));
```



##### props

``` react
//类组件使用props
class Person extends React.Component{
    //对Props属性类型和是否必填进行限制
    static propTypes={
          name:PropTypes.string.isRequired,
          age:PropTypes.num,
          speak:PropTypes.func
      }
      //对标签属性进行默认值限定
      static defaultProps={
          name:"男",
          age:18
      }
    render(){
        return  (
            	<ul>
                  <li>{this.props.name}</li>
                  <li>{this.props.age}</li>
                  <li>{this.props.sex}</li>
              	</ul>
        )
    }
}
function speak(){
    console.log('i am hhl')
}
ReactDOM.render(<Person name="Tom" age="55" sex="男" speak={speak}/>,document.getElementById('test'));
  ReactDOM.render(<Person  age="35" sex="女"/>,document.getElementById('test1'));
  ReactDOM.render(<Person name="Tim" age="64" sex="男"/>,document.getElementById('test2'));
```





``` react
//函数组件中使用props
 function Person(props){
         return (
               <ul>
                   <li>{props.name}</li>
                   <li>{props.age}</li>
                   <li>{props.sex}</li>
               </ul>
           )     
   }
   //只能类外对props的传入进行限制
  Person.propTypes={
           name:PropTypes.string.isRequired,
           age:PropTypes.num,
           speak:PropTypes.func
       }
```



##### ref

``` react
class Demo extends React.Component{
       //调用createRef此API
 	   myRef=React.createRef();
       showinput1=()=>{
           const {input1}=this.refs;
           alert(input1.value);
       }
       showinput2=()=>{
           const {value}=this.myRef.current;
           alert(value);
       }
      render(){
       return (
           <div>
             	 //字符串方式,不推荐，效率较低，从this.refs中取
 			 <input ref="input1" type="text" placeholder="点击按钮显示数据"/>&nbsp;
				//回调函数方式，从this.input1中取
			  <input ref={c=>this.input1=c} type="text" placeholder="点击按钮显示数据"/>
               <button onClick={this.showinput1}>点我显示左边输入框的数据</button>&nbsp;
               <input onBlur={this.showinput2} ref="this.myRef" type="text" placeholder="失去焦点显示数据"/>&nbsp;
           </div>

       )
      }
   }
   ReactDOM.render(<Demo/>,document.getElementById('test'));
```



###### 受控组件实现双向数据绑定

``` react
class Login extends React.Component{
    saveForm=(dataType)=>{
      //高阶函数，及函数柯里化来实现数据统一处理
          return (event)=>{
              this.setState({[dataType]:event.target.value})
              //return的值作为回调函数传给onChange
          }
      }
    state={
        username:'',
        password:''
    }
    loginClick=(event)=>{
        event.preventDefault()
        const {username,password}=this
        alert(username,password)
    }
    render(){
        return (
        <form onSubmit={SubmitClick}>
        	用户名:<input onChange={this.saveForm('username')} type="text" name="username"/>
            //或者使用<input onChange={(e)=>{this.saveForm('username',e)}} type="text" name="username"/>
            密码:<input onChange={this.saveForm('password')} type="password" name="password"/>
            <button onClick={this.loginClick}>登录</button>
        </form>	
        )
    }
}
```



### React生命周期

##### 旧生命周期

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e81e3d01864d4cf487bde1c07ec858b0~tplv-k3u1fbpfcp-watermark.awebp)

##### 新生命周期

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/655312d1e1ba4b0aa477f10ac8375077~tplv-k3u1fbpfcp-watermark.awebp) 

**重要的三个hook**

- render:初始化渲染或更新渲染调用
- componentDidMount:做一些初始化的事情，如：开启定时器，订阅消息、发送Ajax请求
- componentWillUnmount:做一些收尾工作，如：清理定时器

###### 新旧生命周期的区别

- 废弃了3个hooks：ComponentWillMount、ComponentWillReceiveProps、ComponentWillUpdate
- 增加了2个hooks：getDerivedStateFromProps(在state要一直取决于Props的时候可以使用)、getSnapshotBeforeUpdate(用来获取快照来保存更新前的状态 例如：页面的位置)





### key的选取和diff算法

##### 开发如何选择key？

最好使用每一条数据的唯一标识作为key 比如id，手机号，身份证号 如果确定只是简单的展示数据，用Index也是可以的

##### diff算法

Diff算法其实就是react生成的新虚拟DOM和以前的旧虚拟DOM的比较规则：

- 如果旧的虚拟DOM中找到了与新虚拟DOM相同的key并且如果内容没有变化，就直接只用之前旧的真实DOM，如果内容发生了变化，就生成新的真实DOM
- 如果旧的虚拟DOM中没有找到了与新虚拟DOM相同的key，根据数据创建新的真实的DOM,随后渲染到页面上

##### 使用索引值会出现的问题

- 若对数据进行了逆序添加、逆序删除等操作
- 结构中如果存在输入类的DOM，页面有问题



### 代理配置

```react
//在src目录下新建setupProxy.js文件来配置代理
const prox = require('http-proxy-middleware')
module.exports = function(app){
    app.use(
        prox('/api1',{//api1表示含有api1的请求转发
            //转发给localhost:5000
            target:'http://localhost:5000',
            //是否更换源地址，即request中的host
            changeOrigin:true,
            //去除请求前缀，保证请求路径正确
            pathRewrite:{'^/api1':''}
        }),
        prox('/api2',{
            target:'http://localhost:5000',
            changeOrigin:true,
            pathRewrite:{'^/api2':''}
        }),

    )
}
```



### 使用PubSubJs实现消息订阅和发布

*兄弟组件间传统中是通过父组件来实现通信，而现在可以通过PubSub来实现消息订阅和发布*

#### 使用方法

```js
 import PubSub from 'pubsub-js'  //引入PubSub

 PubSub.publish('MY TOPIC', 'hello world!');  //消息发布

 var token = PubSub.subscribe('MY TOPIC', mySubscriber);  //消息订阅
 //mySubscriber是个回调函数 msg是消息描述  data是传输的数据
 var mySubscriber = function (msg, data) {
    console.log( msg, data );
};
PubSub.unsubscribe(token);//订阅取消
```



### Promise补充

###### Promise Api

- Promise.all(promises) —— 等待所有 promise 都 resolve 时，返回存放它们结果的数组。如果给定的任意一个 promise 为 reject，那么它就会变成 Promise.all 的 error，所有其他 promise 的结果都会被忽略。
- Promise.allSettled(promises)（ES2020 新增方法）—— 等待所有 promise 都 settle 时，并以包含以下内容的对象数组的形式返回它们的结果：
- Promise.race(promises) —— 等待第一个 settle 的 promise，并将其 result/error 作为结果。
- Promise.resolve(value) —— 使用给定 value 创建一个 resolved 的 promise。
- Promise.reject(error) —— 使用给定 error 创建一个 rejected 的 promise

###### 中断Promise链的方法

```js
 const p=new Promise((resolve,reject)=>{
            resolve('ok');
        });
        p.then(value=>{
            console.log(111);
            return new Promise(()=>{});  //返回一个pending对象的Promise！
            //否则会返回一个Promise对象 状态由返回值决定
        }).then(value=>{
            console.log(222);
        }).catch(reason=>{
            console.warn(reason);
        })
复制代码
```

###### 使用es7中await/async优化网络请求发送

```js
 try{
        let response =await fetch(url);
        let data=await response.json();
        console.log(data);
    }catch(e){
        console.log("Oops, error",e);
    }
    //外面需要加个 async function
```



### 前端路由

#### 两种前端路由 

- history模式 H5新推出，使用的是H5的history API IE10之后都支持
- hash模式  兼容性良好 特点是path中携带'#' 属于锚点跳转，刷新可能会丢state参数

#### 路由组件

头文件引入 `import {Link,Route} from 'react-router-dom'`

将APP组件包括起来

```js
app.jsx
<div className="list-group">
    //可以用NavLink替换 在点击时会自动加上active类 可以设置activeClassName来设置加的类名
    <Link className="list-group-item"  to="/about">About</Link>
    //设置路由跳转
    <Link className="list-group-item"  to="/home">Home</Link>
</div> 

<div className="panel-body">
    //Routes进行首次匹配
   <Routes>
               <Route path="/about" element={<About/>}></Route>
               <Route path="/home" element={<Home/>}></Route>
   </Routes>
</div>

index.jsx:
ReactDOM.render(
    <BrowserRouter>
        <App/>
    </BrowserRouter>, 
document.getElementById('root'))
```

1.写法不一样

一般组件：< Demo>

路由组件：<Route path="/about" element={<About/>}></Route>

2.存放的位置一般不同

一般组件：components

路由组件：pages

3.接收的内容【props】

一般组件：写组件标签的时候传递什么，就能收到什么

路由组件：接收到三个固定的属性【history,location,match】



tips:可以使用children去获取标签体的内容

##### 多级页面样式丢失

解决方案：

- ./   ->  / 直接在当前目录下匹配
- 使用绝对路径  即 %PUBLIC_URL%
- 使用hashRouter

##### 路由模糊匹配

- 默认使用的是模糊匹配 即 输入路径一定要包含匹配路径(按顺序)
- 开启严格匹配 <Route exact={true} path="/about" element={<About/>}></Route>
- 严格匹配不要随便开启，需要再开，有些时候开启会导致无法继续匹配二级路由

##### Redirect

<Route path="*" element={<Navigate to="/" />}/> 兜底

<Route path='/home/*' element={<Home/>}/>

##### 向路由组件传递参数

**1.params参数**

路由链接（携带参数）

```react
<Link to{'/home/message/detail/${id}/${title}'}>详情 </Link>
```

注册路由（声明接受）:

```react
 <Route path="/home/message/detail/:id/:title" component={Detail} />
```

接受参数:

```react
this.props.match.params
```

**2.search参数**

路由链接（携带参数）

```react
<Link to{'/home/message/detail/?id=${id}/&title=${title}'}>详情 </Link>
```

注册路由（无需声明，正常注册即可）:

```react
 <Route path="/home/message/detail/" component={Detail} />
```

接受参数:

```react
this.props.location.search   //获取到的search是urlencoded编码字符串，需要借助querystring解析
```

**3.state参数**

路由链接（携带参数）

```react
<Link to{{pathname:'/home/message/detail',state:{name:'tom',age:18}}}'}>详情 </Link>
```

注册路由（无需声明，正常注册即可）:

```react
 <Route path="/home/message/detail/" component={Detail} />
```

接受参数:

```react
this.props.location.state
```

路由可以通过开启replace进行无痕浏览 

##### 编程式路由跳转

```react
this.props.history.replace(path)
或
this.props.history.push(path)
```

```react
可以通过 withRouter来加工一般组件来让一般组件具有路由组件所特有的API
```



### Redux

![image-20220213161935699](C:\Users\86159\AppData\Roaming\Typora\typora-user-images\image-20220213161935699.png)

##### Redux异步Action

```react
1.明确：延迟动作不想较给组件自身，想交给action
2.何时需要异步action，想要对状态进行操作，但是具体的数据靠异步任务返回
3.具体编码：
	3.1 npm install redux-thunk 并配置在store中
    3.2 创建action的函数不再返回一般对象，而是一个函数，该函数中写异步任务
    3.3 异步任务有结果后，分发一个同步的action去真正操作数据
4.异步action可以通过在组件内等异步任务来分发同步action
```

### React-redux

![image-20220213172052808](C:\Users\86159\AppData\Roaming\Typora\typora-user-images\image-20220213172052808.png)

## 

##### 理解

- 1. 一个react插件库
- 1. 专门用来简化react应用中使用redux

##### React-redux将组件拆成两大类

- 1. UI组件

  - 1. 只负责 UI 的呈现，不带有任何业务逻辑
  - 1. 通过props接收数据(一般数据和函数)
  - 1. 不使用任何 Redux 的 API
  - 1. 一般保存在components文件夹下

- 1. 容器组件

  - 1. 负责管理数据和业务逻辑，不负责UI的呈现
  - 1. 使用 Redux 的 API
  - 1. 一般保存在containers文件夹下

##### 相关API

1. Provider：让所有组件都可以得到state数据

   ```react
   <Provider store={store}>
     <App />
   </Provider>
   ```

2. connect：用于包装 UI 组件生成容器组件

   ```react
   import { connect } from 'react-redux'
     connect(
       mapStateToprops,
       mapDispatchToProps
     )(Counter)
   ```

3. mapStateToprops：将外部的数据（即state对象）转换为UI组件的标签属性

   ```react
   const mapStateToprops = function (state) {
     return {
       value: state
     }
   }
   ```

4. mapDispatchToProps：将分发action的函数转换为UI组件的标签属性

##### react-redux相较于redux的优势

- 可以使项目可拓展性和可维护性更强
- 因为使用了connect函数，所以自动会监听store是否有更新
- 可以使用Provider来自动管理store 
- 多个reducer时要用combineReducers

##### 纯函数

- react-reducer中必须是纯函数
- 纯函数不能调用Math.random、Date.now 之类的函数
- 纯函数里不能有发送网络请求、输入输出设备等
- 纯函数不得改写参数数据

##### react-devtools

```react
npm install --save-dev redux-devtools-extension
```

```react
import {composeWithDevTools} from 'redux-devtools-extension'
const store = create(allReducer,composeWithDevTools(applyMiddleware(thunk)))
```



### React补充

#### hooks 存在的意义

1. hooks 之间的状态是独立的，有自己独立的上下文，不会出现混淆状态的情况
2. 让函数有了状态管理
3. 解决了 组件树不直观、类组件难维护、逻辑不易复用的问题
4. 避免函数重复执行的副作用

#### 应用场景

1. 利用 hooks 取代生命周期函数
2. 让组件有了状态
3. 组件辅助函数
4. 处理发送请求
5. 存取数据
6. 做好性能优化

#### hooks API

从 `react` 中引入

##### 1. useState

给函数组件添加状态

- 初始化以及更新组件状态

```
const [count, setCount] = React.useState(0)
```

接收一个参数作为初始值，返回一个数组：第一个是状态变量，第二个是修改变量的函数

##### 2. useEffect

副作用 hooks

- 给没有生命周期的组件，添加结束渲染的信号

注意：

- render 之后执行的 hooks

第一个参数接收一个函数，在组件更新的时候执行

第二个参数接收一个数组，用来表示需要追踪的变量，依赖列表，只有依赖更新的时候才会更新内容

第一个参数的返回值，返回一个函数，在 `useEffect` 执行之前，都会先执行里面返回的函数

一般用于添加销毁事件，这样就能保证只添加一个

```
React.useEffect(() => {
    console.log('被调用了');
    return () => {
        console.log('我要被卸载了');
    }
}, [count])
```

打印

[![image-20210914172936843](https://camo.githubusercontent.com/cf51ba421a1f83d55be800fde74c50176acc8ad142f0324648e0cd39d2aed0e8/68747470733a2f2f6c6a63696d672e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f696d672f696d6167652d32303231303931343137323933363834332e706e67)](https://camo.githubusercontent.com/cf51ba421a1f83d55be800fde74c50176acc8ad142f0324648e0cd39d2aed0e8/68747470733a2f2f6c6a63696d672e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f696d672f696d6167652d32303231303931343137323933363834332e706e67)

### 3. useLayoutEffect

和 `useEffect` 很类似

它的作用是：在 DOM 更新完成之后执行某个操作

注意：

- 有 DOM 操作的副作用 hooks
- 在 DOM 更新之后执行

> 执行时机在 `useEffect` 之前，其他都和 `useEffect` 都相同

`useEffect` 执行时机在 **render 之后**

`useLayoutEffect` 执行时机在 **DOM 更新之后**

##### 4. useMemo

作用：让组件中的函数跟随状态更新

注意：优化函数组件中的功能函数

**为了避免由于其他状态更新导致的当前函数的被迫执行**

第一个参数接收一个函数，第二个参数为数组的依赖列表，返回一个值

```
const getDoubleNum = useMemo(() => {
    console.log('ddd')
    return 2 * num
}, [num])
```

###### 5. useCallback

作用：跟随状态更新执行

注意：

- 只有依赖项改变时才执行
- `useMemo( () => fn, deps)` 相当于 `useCallback(fn, deps)`

不同点：

1. `useCallback` **返回的是一个函数，不再是值**
2. `useCallback` 缓存的是一个函数，`useMemo` 缓存的是一个值，**如果依赖不更新，返回的永远是缓存的那个函数**
3. 给子组件中传递 `props` 的时候，如果当前组件不更新，不会触发子组件的重新渲染

##### 6. useRef

作用：长久保存数据

注意事项：

- 返回一个子元素索引，这个索引在整个生命周期中保持不变
- 对象发生改变时，不通知，属性变更不重新渲染

1. 保存一个值，在整个生命周期中维持不变
2. 重新赋值 `ref.current` 不会触发重新渲染
3. 相当于**创建一个额外的容器来存储数据**，我们可以在外部拿到这个值

当我们通过正常的方式去获取计时器的 `id` 是无法获取的，需要通过 `ref`

```
useEffect(() => {
    ref.current = setInterval(() => {
        setNum(num => num + 1)
    }, 400)
}, [])
useEffect(() => {
    if (num > 10) {
        console.log('到十了');
        clearInterval(ref.current)
    }
}, [num])
```

##### 7. useContext

作用：带着子组件渲染

注意：

- 上层数据发生改变，肯定会触发重新渲染

1. 我们需要引入 `useContext` 和 `createContext` 两个内容
2. 通过 `createContext` 创建一个 `Context` 句柄
3. 通过 `Provider` 确定数据共享范围
4. 通过 `value` 来分发数据
5. 在子组件中，通过 `useContext` 来获取数据

```
import React, { useContext, createContext } from 'react'
const Context = createContext(null)
export default function Hook() {
    const [num, setNum] = React.useState(1)

    return (
        <h1>
            这是一个函数组件 - {num}
        // 确定范围
            <Context.Provider value={num}>
                <Item1 num={num} />
                <Item2 num={num} />
            </Context.Provider>
        </h1>
    )
}
function Item1() {
    const num = useContext(Context)
    return <div>子组件1  {num}</div>
}
function Item2() {
    const num = useContext(Context)
    return <div>子组件2 {num}</div>
}
```

##### 8. useReducer

作用：去其他地方借资源

注意：函数组件的 Redux 的操作

1. 创建数据仓库 `store` 和管理者 `reducer`
2. 通过 `useReducer(store,dispatch)` 来获取 `state` 和 `dispatch`

```
const store = {
    num: 10
}
const reducer = (state, action) => {
    switch (action.type) {
        case "":
            return
        default:
            return
    }
}
    const [state, dispatch] = useReducer(reducer, store)
```

通过 `dispatch` 去派发 `action`

##### 9. 自定义 hooks

放在 `utils` 文件夹中，以 `use` 开头命名

例如：模拟数据请求的 Hooks

```
import React, { useState, useEffect } from "react";
function useLoadData() {
  const [num, setNum] = useState(1);
  useEffect(() => {
    setTimeout(() => {
      setNum(2);
    }, 1000);
  }, []);
  return [num, setNum];
}
export default useLoadData;
```

减少代码耦合

我们希望 reducer 能让每个组件来使用，我们自己写一个 hooks

自定义一个自己的 LocalReducer

```
import React, { useReducer } from "react";
const store = { num: 1210 };
const reducer = (state, action) => {
  switch (action.type) {
    case "num":
      return { ...state, num: action.num };
    default:
      return { ...state };
  }
};
function useLocalReducer() {
  const [state, dispatch] = useReducer(reducer, store);
  return [state, dispatch];
}
export default useLocalReducer;
```

1. 引入 react 和自己需要的 hook
2. 创建自己的hook函数
3. 返回一个数组，数组中第一个内容是数据，第二个是修改数据的函数
4. 暴露自定义 hook 函数出去
5. 引入自己的业务组件



## 动态添加样式和类名

react开发过程中，经常会需要动态向元素内添加样式style或className，那么应该如何动态添加呢？？？

一、react向元素内，动态添加style

例如：有一个DIV元素， 需要动态添加一个 display:block | none 样式， 那么：

<div style={{display: (index===this.state.currentIndex) ? "block" : "none"}}>此标签是否隐藏</div>
或者， 多个样式写法：

<div style={{display: (index===this.state.currentIndex) ? "block" : "none", color:"red"}}>此标签是否隐藏</div>
二、react向元素内，动态添加className

比如：

1、DIV标签中，没有其他class，只需要动态添加一个.active的className，来显示内容是否被选中状态，则：

<div className={index===this.state.currentIndex?"active":null}>此标签是否选中</div>
2、如果DIV标签本身有其他class，又要动态添加一个.active的className，来显示内容是否被选中状态，则：

<div className={["container tab", index===this.state.currentIndex?"active":null].join(' ')}>此标签是否选中</div>
或者，使用ES6写法（推荐使用ES6写法）：

<div className={`container tab ${index===this.state.currentIndex?"active":null}`}>此标签是否选中</div>

