### 前端算法

##### [剑指 Offer II 087. 复原 IP ](https://leetcode-cn.com/problems/0on3uN/)

```js
var restoreIpAddresses = function(s) {
    let l=s.length
    let ans=[]
    //i,j,k表示.分割的位置
    for(let i=1;i<4;i++){
        for(let j=i+1;j<i+4;j++){
            for(let k=j+1;k<j+4;k++){
                let a=s.slice(0,i)
                let b=s.slice(i,j)
                let c=s.slice(j,k)
                let d=s.slice(k,l)
                if(check(a)&&check(b)&&check(c)&&check(d)){
                    ans.push(a+'.'+b+'.'+c+'.'+d)
                }
            }
        }
    }
    function check(str){
        let num=parseInt(str)
        //如果转成num后再转回字符串不一样 说明可能是0开头的
        if(num+''!=str) return false
        if(num>=0&&num<=255) return true
        return false
    }
    return ans
};
```





##### 大数相加

```js
function add(a,b){
    let maxLength = Math.max(a.length,b.length)
    a = a.padStart(maxLength,0)
    b = b.padStart(maxLength,0)
    let f = 0 //进位
    let sum
    let ans = ''
    for(let i=maxLength-1;i>=0;i--){
        sum = parseInt(a[i]) + parseInt(b[i]) + f
        f = Math.floor(sum / 10)
        ans = sum%10 + ans 
    }
    if(f == 1){
        sum = '1' + sum
    }
    return ans
}
```









### 前端手写





##### 数据类型判断

```js
function myTypeOf(obj){
        return Object.prototype.toString.call(obj).slice(8,-1).toLowerCase()
}
```



##### 判断一个对象是否为空对象

```js
function checkNullObj(obj){
    return object.keys(obj).length === 0 && Object.getOwnPropertySymbols(obj).length === 0
}
```



##### 实现每隔一秒打印 1,2,3,4

```js
// 使用闭包实现
for (var i = 0; i < 5; i++) {
  (function(i) {
    setTimeout(function() {
      console.log(i);
    }, i * 1000);
  })(i);
}

// 使用 let 块级作用域
for (let i = 0; i < 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, i * 1000);
}
```



#####  js 中倒计时的纠偏实现？

```js
const interval = 1000
let ms = 50000,  // 从服务器和活动开始时间计算出的时间差，这里测试用 50000 ms
let count = 0
const startTime = new Date().getTime()
let timeCounter
if( ms >= 0) {
  timeCounter = setTimeout(countDownStart, interval)
}
 
function countDownStart () {
   count++
   const offset = new Date().getTime() - (startTime + count * interval) // A
   let nextTime = interval - offset
   if (nextTime < 0) { 
       nextTime = 0 
   }
   ms -= interval
   console.log(`误差：${offset} ms，下一次执行：${nextTime} ms 后，离活动开始还有：${ms} ms`)
   if (ms < 0) {
     clearTimeout(timeCounter)
   } else {
     timeCounter = setTimeout(countDownStart, nextTime)
   }
 }
```







##### 如何测试localStroage的大小

```js
let total = ''
  let str = ''
  while (str.length !== 10240) {
  str = str + '0123456789'
  }
  localStorage.clear()
  function testLocalStorage(){
    let timer = setInterval(()=>{
      try {
        localStorage.setItem('test',total)
      } catch (e) {
        clearInterval(timer)
        console.log(`爆满了,最大值为${total.length/1024}KB`);
        localStorage.clear()
      }
      total += str
    },0)
  }
  testLocalStorage()
```



### 循环打印红绿灯(状态模式)

[掘金状态模式好文](https://juejin.cn/post/7088863590375456781#heading-5)

```js
let state = [
    {
        color:'red',
        time:3000
    },
    {
        color:'yellow',
        time:2000
    },
    {
        color:'green',
        time:5000
    }
]

let total = state.reduce((prev,curr)=>prev+curr.time,0)
async function startShow(){
    for(let i=0;i<state.length;i++){
        console.log(state[i].color);
        await new Promise((resolve)=>{
            setTimeout(() => {
                resolve()
            }, state[i].time);
        })
    }
}

startShow()
setInterval(startShow,total)
```





### 数组转树形 

```js
// 期望结果
[
  {
    "id": 1,
    "pid": 0,
    "text": "xxx",
    "children": [
      {
        "id": 2,
        "pid": 1,
        "text": "xxx-2",
        "children": [
          {
            "id": 4,
            "pid": 2,
            "text": "xxx-2-4"
          },
          {
            "id": 6,
            "pid": 2,
            "text": "xxx-2-6"
          }
        ]
      },
      {
        "id": 3,
        "pid": 1,
        "text": "xxx-3",
        "children": [
          {
            "id": 5,
            "pid": 3,
            "text": "xxx-3-5"
          }
        ]
      }
    ]
  }
]

let data = [
    {id: 1, pid: 0, text: 'xxx'},
    {id: 2, pid: 1, text: 'xxx-2'},
    {id: 3, pid: 1, text: 'xxx-3'},
    {id: 4, pid: 2, text: 'xxx-2-4'},
    {id: 5, pid: 3, text: 'xxx-3-5'},
    {id: 6, pid: 2, text: 'xxx-2-6'}
];

let obj ={}
for(let i=0;i<data.length;i++){
    let item = data[i]
    obj[item.id] = item
}
let res = null
for(let i=0;i<data.length;i++){
    let item = data[i]
    if(item.pid == 0){
        res = item
    }else{
        if(obj[item.pid].children){
            obj[item.pid].children.push(item)
        }else{
            obj[item.pid].children = [item]
        }
    }
}
console.log(res);
```



### 树转数组

```js
const root = [
    {
        id: 1,
        name: '1x',
        children: [
            {
                id: 11, name: '11x', children: [{ id: 111, name: '111x' }]
            },
            { id: 12, name: '12x' },
        ]
    },
    {
        id: 2,
        name: '1x',
        children: [
            {
                id: 21, name: '21x', children: [{ id: 211, name: '21x' }]
            },
            { id: 12, name: '12x' },
        ]
    },
    {id: 3,name: '3x'}
]
//输出以下格式，
//{id:1,name:'1x',children:[11,21]}
//{id:11,name:'1x',parent:[1],children:[111]}
//{id:12,name:'12x',parent:[1]}
//{id:3,name:'3x'}

let res = {}
dfs(root)
function dfs(nodes=[],pid=null){
  let children = []
  for(let i=0;i<nodes.length;i++){
    let id = nodes[i].id
    children.push(id)
    if(res[id] === undefined){
      res[id] = {id,name:nodes[i].name}
    }
    if(pid){
      res[id].parent = [pid]
    }
    let myChildren = dfs(nodes[i].children,id)
    if(myChildren.length>0){
      nodes[i].children = myChildren
    }
  }
  return children
}
console.log(res);
```



树的复选框改变勾选状态

```js
/**
 * 题目：请根据传入的勾选项数组，完成数据勾选状态标记
 *
 * 补充：允许修改原对象，允许增添字段辅助完成功能
 *
 * 用例 1
 * 输入： [unmarkData, ['1-1'，‘1-1-1’]]
 * 输出： [
 *  {
 *    id: '1',
 *    name: '组织1',
 *    status: checkStatus.all,
 *    children: [
 *      {
 *        id: '1-1',
 *        name: '组织1 - 子组织1',
 *        status: checkStatus.all,
 *        children: [
 *          {
 *            id: '1-1-1',
 *            name: '组织1 - 子组织1 - 孙子组织1',
 *            status: checkStatus.all,
 *          },
            {
 *            id: '1-1-2',
 *            name: '组织1 - 子组织1 - 孙子组织1',
 *            status: checkStatus.all,
 *          },
 *        ],
 *      },
 *      {
 *        id: '1-2',
 *        name: '组织1 - 子组织2',
 *        status: checkStatus.all,
 *      },
 *    ],
 *  },
 *]
 *
 * 用例 2
 * 输入： [unmaskData, ['1-1-1']]
 * 输出： [
 *  {
 *    id: '1',
 *    name: '组织1',
 *    status: checkStatus.part,
 *    children: [
 *      {
 *        id: '1-1',
 *        name: '组织1 - 子组织1',
 *        status: checkStatus.all,
 *        children: [
 *          {
 *            id: '1-1-1',
 *            name: '组织1 - 子组织1 - 孙子组织1',
 *            status: checkStatus.all,
 *          },
 *        ],
 *      },
 *      {
 *        id: '1-2',
 *        name: '组织1 - 子组织2',
 *        status: checkStatus.none,
 *      },
 *    ],
 *  },
 *]
 *
 */

/** 勾选状态定义 */
const checkStatus = {
    /** 表示当前节点的所有子节点（包括子节点中的子节点等等）均未被勾选 */
    'none' : 0,
    /** 表示当前节点的所有子节点（包括子节点中的子节点等等）部分被勾选 */
    'part' : 1,
    /** 表示当前节点的所有子节点（包括子节点中的子节点等等）全部被勾选 */
    'all' : 2,
  }
  
  /** 需要处理的数据，默认 id 唯一，深度与广度不确定 */
  const unmarkData = [
    {
      id: '1',
      name: '组织1',
      status: checkStatus.none,
      children: [
        {
          id: '1-1',
          name: '组织1 - 子组织1',
          status: checkStatus.none,
          children: [
            {
              id: '1-1-1',
              name: '组织1 - 子组织1 - 孙子组织1',
              status: checkStatus.none,
            },
          ],
        },
        {
          id: '1-2',
          name: '组织1 - 子组织2',
          status: checkStatus.none,
        },
      ],
    },
  ]
  
  function mark(unmarkData, markIDs) {
      dfs(unmarkData)
      function dfs(nodes){
          let children = []
          for(let i=0;i<nodes.length;i++){
              let node = nodes[i]
              if(markIDs.includes(node.id)){
                  children.push(node.id)
                  node.status = checkStatus.all
                  let arr = node.children 
                  let index = 0
                  while(index<arr.length){
                      let temp = arr[index]
                      index++
                      temp.status = checkStatus.all
                      if(temp.children){
                          arr.push(...temp.children)
                      }
                  }
              }
              let mychildren = []
              if(node.children){
                  mychildren = dfs(node.children)
              }
              if(mychildren.length == node.children){
                  node.status = checkStatus.all
              }else if(mychildren.length>0){
                  node.status = checkStatus.part
              }
          }
          return children
      }
  }
markIDs = ['1-1']
mark(unmarkData,markIDs)
console.dir(unmarkData)
  
```











##### 继承

###### 原型链继承

```js
function Animal() {
    this.colors = ['black', 'white']
}
Animal.prototype.getColor = function() {
    return this.colors
}
function Dog() {}
Dog.prototype =  new Animal()

let dog1 = new Dog()
dog1.colors.push('brown')

let dog2 = new Dog()
console.log(dog2.colors)  // ['black', 'white', 'brown']

//问题1：原型中包含的引用类型属性将被所有实例共享； 因为Dog.prototype =  new Animal() 只有一个animal实例共享
//问题2：子类在实例化的时候不能给父类构造函数传参；

```





###### 借用构造函数实现继承

```js
function Animal(name){
    this.name=name
    this.getName=function(){
      return this.name
    }
}
//这样是方法绑定在原型对象上 上面是绑定在实例上
// Animal.prototype.getName=function(){
  //     return this.name
  //   }

function Dog(name){
    Animal.call(this,name)
}
let dog1=new Dog('鲁棒杰')
console.log(dog1.getName());
//借用构造函数实现继承解决了原型链继承的 2 个问题：引用类型共享问题以及传参问题。
//但是由于方法必须定义在构造函数中，所以会导致每次创建子类实例都会创建一遍方法。
```





###### 组合继承

```js
function Animal(name){
        this.name=name
        this.colors=['black','white']
}
Animal.prototype.getName=function(){
    return this.name
}
function Dog(name,age){
    Animal.call(this,name)
    this.age=age
}
Dog.prototype=new Animal()
Dog.prototype.constructor =Dog
let dog1=new Dog('LBJ',12)
dog1.colors.push('brown')
let dog2=new Dog('CSY',14)
console.log(dog2);
console.log(dog2.getName());
//组合继承结合了原型链和盗用构造函数，将两者的优点集中了起来
```



###### 原型式继承

```js
function createAnother(original){
    let F=function(){}
    F.prototype=original
    return new F()
}
```



###### 寄生虫继承

```js
function createAnother(original){
    let newObj=createAnother(original)
    newObj.prototype.add = function(){...}
    return newObj
}
```



###### 寄生式组合继承

```js
function Animal(name){
        this.name=name
        this.colors=['black','white']
}
Animal.prototype.getName=function(){
    return this.name
}
function Dog(name,age){
    Animal.call(this,name)
    this.age=age
}
// - Dog.prototype=new Animal()
// - Dog.prototype.constructor = Dog
	Dog.prototype =  Object.create(Animal.prototype)
	Dog.prototype.constructor = Dog

let dog1=new Dog('LBJ',12)
dog1.colors.push('brown')
let dog2=new Dog('CSY',14)
console.log(dog2);
console.log(dog2.getName());
```



###### ES6继承

```js
function Animal(name){
    constructor(name) {
        this.name = name
    } 
    getName() {
        return this.name
    }
}
class Dog extends Animal {
    constructor(name, age) {
        super(name)
        this.age = age
    }
}
```





### 类数组如何转换成数组

（1）通过 call 调用数组的 slice 方法来实现转换

```
Array.prototype.slice.call(arrayLike);  [].slice.call(arugments)
```

（2）通过 call 调用数组的 splice 方法来实现转换

```
Array.prototype.splice.call(arrayLike, 0);
```

（3）通过 apply 调用数组的 concat 方法来实现转换

```
Array.prototype.concat.apply([], arrayLike);
```

（4）通过 Array.from 方法来实现转换

```
Array.from(arrayLike);
```

（5）

```js
[...arguments]
```

(6)

```js
function(...args){
    //此时args就是已经转换的arguments数组
}
```















##### 数组去重

```js
//es6
function unique(arr){
    return Array.from(new Set(arr))
}
//es5
function unique(arr){
    return arr.filter((item,index)=>{
        return arr.indexOf(item)==index
    })
}
//双层for + splice
function unique(arr){
    for(let i=0;i<arr.length;i++){
        for(let j=i+1;j<arr.length;j++){
            if(arr[j]==arr[i]){
                arr.splice(j,1)
                j--
            }
        }
    }
    return arr
}
//新创一个数组
function unique(arr){
    let res = []
    for(let i=0;i<arr.length;i++){
        if(!res.includes(arr[i])){
            res.push(arr[i])
        }
    }
    return res
}
//利用对象的key只能唯一
function unique(arr){
    let obj={}
    let res = []
    for(let i=0;i<arr.length;i++){
        if(!obj[arr[i]]){
            res.push(arr[i])
            obj[arr[i]] = 1
        }
    }
    return res
}
console.log(unique([1,2,3,4,4,4,4]));
```



##### 数组扁平化

```js
//递归 es5
function myFlat(arr){
    let res=[]
    for(let i=0;i<arr.length;i++){
        let item=arr[i]
        if(Array.isArray(item)){
            res=res.concat(myFlat(item))
        }else{
            res.push(item)
        }
    }
    return res
}
//es6 循环
function myFlat(arr){
    while(arr.some(item=>Array.isArray(item))){
        arr=[].concat(...arr)
    }
    return arr
}
console.log(myFlat([1,2,3,[2,3,4],[2,[2]]]));
```



##### 深浅拷贝

```js
const isObject = (target) => (typeof target === "object" || typeof target === "function") && target !== null;

function deepClone(target, map = new WeakMap()) {
    if (map.get(target)) {
        return target;
    }
    // 获取当前值的构造函数：获取它的类型
    let constructor = target.constructor;
    // 检测当前对象target是否与正则、日期格式对象匹配
    if (/^(RegExp|Date)$/i.test(constructor.name)) {
        // 创建一个新的特殊对象(正则类/日期类)的实例
        return new constructor(target);  
    }
    if (isObject(target)) {
        map.set(target, true);  // 为循环引用的对象做标记
        const cloneTarget = Array.isArray(target) ? [] : {};
        for (let prop in target) {
            if (target.hasOwnProperty(prop)) {
                cloneTarget[prop] = deepClone(target[prop], map);
            }
        }
        return cloneTarget;
    } else {
        return target;
    }
}
//常用的库有 loadsh中 _.cloneDeep() 
// jQuery.extend()和JSON.stryify()

```



##### 事件总线（发布订阅模式）

```js
class EventEmitter {
    constructor() {
        this.cache = {}
    }
    on(name, fn) {
        if (this.cache[name]) {
            this.cache[name].push(fn)
        } else {
            this.cache[name] = [fn]
        }
    }
    off(name, fn) {
        let tasks = this.cache[name]
        if (tasks) {
            const index = tasks.findIndex(f => f === fn || f.callback === fn)
            if (index >= 0) {
                tasks.splice(index, 1)
            }
        }
    }
    emit(name, once = false, ...args) {
        if (this.cache[name]) {
            // 创建副本，如果回调函数内继续注册相同事件，会造成死循环
            let tasks = this.cache[name].slice()
            for (let fn of tasks) {
                fn(...args)
            }
            if (once) {
                delete this.cache[name]
            }
        }
    }
}

// 测试
let eventBus = new EventEmitter()
let fn1 = function(name, age) {
    console.log(`${name} ${age}`)
}
let fn2 = function(name, age) {
    console.log(`hello, ${name} ${age}`)
}
eventBus.on('aaa', fn1)
eventBus.on('aaa', fn2)
eventBus.emit('aaa', false, '布兰', 12)
// '布兰 12'
// 'hello, 布兰 12'

```









##### 手写观察者模式

```js
class Subject { //被观察者数据---吊瓶
    constructor(name="知了哥"){
        this.state = 100;
        this.name = name;
        this.obs = [] //会有多个观察者
    }
    addObs(ob){
        this.obs.push(ob)
    }
    setState(state){ //改变状态的方法
        this.state = state
        //要去通知观察者去做动作，--- 拔针 
        this.obs.forEach((ob)=>{
            ob.update(this) //让观察者去做更新
        })
    }
}

class Obeserver{ //观察者 -- 医生

    constructor(name){
        this.name = name;
        //在观察者里面是不是也可以记录 观察了哪些 数据（吊瓶）
    }
    update(subject){
        //医生也有可能观察多个 打针的人 
        if (!subject.state) {
            console.log(`${this.name} 收到通知 :${subject.name} 的 带瓶打完啦！`)
        }else {
            console.log(` ${this.name} 收到通知 :${subject.name} 的 带瓶量: ${subject.state}！`)
        }

    }

}
var zhiliao = new Subject();
var hushi = new Obeserver("护士");
var yisheng = new Obeserver("医生")
zhiliao.addObs(hushi) //把观察者放到被观察这的里面去
zhiliao.addObs(yisheng)
zhiliao.setState(50) 
```









##### 解析URL参数为对象

```js
function parseParam(url) {
    const paramsStr = /.+\?(.+)$/.exec(url)[1]; // 将 ? 后面的字符串取出来
    const paramsArr = paramsStr.split('&'); // 将字符串以 & 分割后存到数组中
    let paramsObj = {};
    // 将 params 存到对象中
    paramsArr.forEach(param => {
        if (/=/.test(param)) { 
            let [key, val] = param.split('='); 
            val = decodeURIComponent(val); 
            val = /^\d+$/.test(val) ? parseFloat(val) : val; // 判断是否转为数字

            if (paramsObj.hasOwnProperty(key)) { // 如果对象有 key，则添加一个值
                paramsObj[key] = [].concat(paramsObj[key], val);
                //这里使用[].concat的原因是需要合并一个数组和一个单独的值 使用[].concat很合适
            } else { 
                paramsObj[key] = val;
            }
        } else { // 处理没有 value 的参数
            paramsObj[param] = true;
        }
    })
    return paramsObj;
}

```



##### 字符串模板

```js
function render(template, data) {
    const reg = /\{\{(\w+)\}\}/; // 模板字符串正则
    if (reg.test(template)) { // 判断模板里是否有模板字符串
        const name = reg.exec(template)[1]; // 查找当前模板里第一个模板字符串的字段
        template = template.replace(reg, data[name]); // 将第一个模板字符串渲染
        return render(template, data); // 递归的渲染并返回渲染后的结构
    }
    return template; // 如果模板没有模板字符串直接返回
}

```



##### 图片懒加载

```js
let imgList = [...document.querySelectorAll('img')]
let length = imgList.length

const imgLazyLoad = (function() {
    let count = 0
    
   return function() {
        let deleteIndexList = []
        imgList.forEach((img, index) => {
            let rect = img.getBoundingClientRect() 
            // https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect
            if (rect.top < window.innerHeight) {
                img.src = img.dataset.src
                deleteIndexList.push(index)
                count++
                if (count === length) {
                    document.removeEventListener('scroll', imgLazyLoad)
                }
            }
        })
        imgList = imgList.filter((img, index) => !deleteIndexList.includes(index))
   }
})()

// 这里最好加上防抖处理
document.addEventListener('scroll', imgLazyLoad)

```



##### 函数防抖

```js
function debounce(fn,wait){
    let timeout
    return function(){
        let context = this
        let args = arguments
        clearTimeout(timeout)
        timeout=setTimeout(()=>{
            fn.call(context,args)
        },wait)
    }
}

//完整版
function debounce(func, wait, immediate) {
    var timeout, result;

    var debounced = function () {
        var context = this;
        var args = arguments;

        if (timeout) clearTimeout(timeout);
        if (immediate) {
            // 如果已经执行过，不再执行
            var callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            }, wait)
            if (callNow) result = func.apply(context, args)
        } else {
            timeout = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
        return result;
    };

    debounced.cancel = function() {
        clearTimeout(timeout);
        timeout = null;
    };

    return debounced;
}

```





##### 函数节流

最终版：支持取消节流；另外通过传入第三个参数，options.leading 来表示是否可以立即执行一次，opitons.trailing 表示结束调用的时候是否还要执行一次，默认都是 true。 注意设置的时候不能同时将 leading 或 trailing 设置为 false。

```js
function throttle(func, wait){
    let prev = 0	
    return function(){
        args = arguments
        let now = +new Date()
        if(now - prev > wait){
            func.call(context,args)
            prev=now
        }
    }
}

//完整版
function throttle(func, wait, options){
    let timeout,context,args,result
    let previous=0
    if(!options) options={}
    let later=function (){
        previous
    }

    let throttled= function(){
        let now=new Date().getTime()
        //如果首次不执行的情况时 previous就初始化为now 这样就不会首次执行了
        if(!previous && options.leading === false) previous = now
        let remaining =wait -(now - previous)
        context=this
        args=arguments
        //如果到时间了 1.超时了 2.首次
        if(remaining<=0 ||remaining >wait){
            if(timeout){
                clearTimeout(timeout)
                timeout=null
            }
            func.apply(context,args)
            if(!timeout) context = args = null
        }else if(!timeout && options.trailing !== false){
            timeout = setTimeout(later,remaining)
        }
    }

    throttle.cancel=function(){
        clearTimeout(timeout)
        previous = 0
        timeout =null
    }
    return throttled
}
```



##### 函数柯里化

```js
function curry(fn) {
    let judge = (...args) => {
        if (args.length == fn.length) return fn(...args)
        return (...arg) => judge(...args, ...arg)
    }
    return judge
}
//es6 简单写法
function curry(fn,...args){
    return fn.length == args.length ? fn(...args) : curry.bind(null,fn,...args)
}
```



##### 偏函数

```js
function partial(fn,...args){
    return (...arg)=>{
        return fn(...args,...arg)
    }
}
```



##### JSONP

```js
const jsonp = ({ url, params, callbackName }) => {
    const generateUrl = () => {
        let dataSrc = ''
        for (let key in params) {
            if (params.hasOwnProperty(key)) {
                dataSrc += `${key}=${params[key]}&`
            }
        }
        dataSrc += `callback=${callbackName}`
        return `${url}?${dataSrc}`
    }
    return new Promise((resolve, reject) => {
        const scriptEle = document.createElement('script')
        scriptEle.src = generateUrl()
        document.body.appendChild(scriptEle)
        window[callbackName] = data => {
            resolve(data)
            document.removeChild(scriptEle)
        }
    })
}

```



##### AJAX

```js	
const getJSON = function(url) {
    return new Promise((resolve, reject) => {
        const xhr = XMLHttpRequest ? new XMLHttpRequest() : new ActiveXObject('Mscrosoft.XMLHttp');
        xhr.open('GET', url, false);
        xhr.setRequestHeader('Accept', 'application/json');
        xhr.onreadystatechange = function() {
            if (xhr.readyState !== 4) return;
            if (xhr.status === 200 || xhr.status === 304) {
                resolve(xhr.responseText);
            } else {
                reject(new Error(xhr.responseText));
            }
        }
        xhr.send();
    })
}

```



##### 实现数组的原型方法

###### forEach

```js
Array.prototype.forEach2 = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or not defined')
    }
    if (typeof callback !== "function") {
        throw new TypeError(callback + ' is not a function')
    }
    const O = Object(this)  // this 就是当前的数组
    const len = O.length >>> 0  // 后面有解释
    let k = 0
    while (k < len) {
        if (k in O) {
            callback.call(thisArg, O[k], k, O);
        }
        k++;
    }
}

```



###### map

```js
  Array.prototype.map2 = function(callback, thisArg) {
    if (this == null) {
        throw new TypeError('this is null or not defined')
    }
    if (typeof callback !== "function") {
        throw new TypeError(callback + ' is not a function')
    }
    const O = Object(this)
    const len = O.length >>> 0
    let k = 0, res = []
    while (k < len) {
        if (k in O) {
            res[k] = callback.call(thisArg, O[k], k, O);
        }
        k++;
    }
    return res
}

```



###### filter

```js
 Array.prototype.filter2 = function(callback, thisArg) {
    if (this == null)  throw new TypeError('this is null or not defined')
    if (typeof callback !== "function")  throw new TypeError(callback + ' is not a function')
    const O = Object(this)
    const len = O.length >>> 0
    let k = 0, res = []
    while (k < len) {
        if (k in O) {
           if (callback.call(thisArg, O[k], k, O)) {
               res.push(O[k])                
           }
        }
        k++;
    }
   return res
}
```



###### some

```js
Array.prototype.some1 = function (callback,thisArg){
    if(this == null) throw new TypeError('this is null or not defined')
    if(typeof callback !=='function') throw new TypeError(callback + 'is not a function')
    let O = Object(this)
    let len= O.length >>>0
    let k=0
    while(k<len){
        if(k in O){
            if(callback(thisArg,O[k],k,O)){
                return true
            }
        }
        k++
    }
    return false
}
```



###### reduce

```js
Array.prototype.reduce2 = function (callback, initalValue){
    if(this == null) throw new TypeError('this is null or not defined')
    if(typeof callback !== 'function') throw new Error(callback + 'is not a function')
    let acc
    let k=0
    let O = Object(this)
    let len = O.length >>> 0
    if(initalValue){
        acc=initalValue
    }else{
        while(k<len && !(k in O)){
            k++
        }
        if(k>=len){
            throw new TypeError('Reduce of empty array with no initial value')
        }
        acc = O[k++]
    }
    while(k<len){
        if(k in O){
            acc = callback(acc,O[k],k,O)
        }
        k++
    }
    return acc
}
```



##### 实现函数原型方法

###### call

```js
Function.prototype.myCall = function(context = window){ //myCall函数的参数，没有传参默认是指向window
  if (typeof this !== "function") {
    console.error("type error");
  }
  context.fn = this //为对象添加方法（this指向调用myCall的函数）
  let args = [...arguments].slice(1) // 剩余的参数 
  let res = context.fn(...args)  // 调用该方法，该方法this指向context
  delete context.fn //删除添加的方法
  return res
}
```



###### apply

```js
Function.prototype.myApply = function(context = window){ //myCall函数的参数，没有传参默认是指向window
  if (typeof this !== "function") {
    console.error("type error");
  }
  context.fn = this //为对象添加方法（this指向调用myCall的函数）
  let res
  if(arguments[1]){ //判断是否有第二个参数
    res = context.fn(...arguments[1])// 调用该方法，该方法this指向context
  }else{
    res = context.fn()// 调用该方法，该方法this指向context
  }
  delete context.fn //删除添加的方法
  return res
}
```



###### bind

```js	
Function.prototype.myBind = function(context = window){
  if (typeof this !== "function") {
    console.error("type error");
  }
  let fn = this // 调用bind的函数
  let args = [...arguments].slice(1) // myBind的参数
  let bind = function(){	
    let args1 = [...arguments].slice() // bind的参数
    return fn.apply(context,args.concat(args1))
  }
return bind
}
```



##### 实现new关键字

```js
function objectFactory() {
    var obj = new Object()
    Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    var ret = Constructor.apply(obj, arguments);
    // ret || obj 这里这么写考虑了构造函数显示返回 null 的情况
    return typeof ret === 'object' ? ret || obj : obj;
};

```



##### 实现instanceof

```js
function instanceOf (left,right){
    let proto = left.__proto__
    while(true){
        if(proto==null) return false
        if(proto==right.prototype) return true
        proto=proto.__proto__
    }
}
```



##### 实现Object.create

```js
Object.create2 = function(proto, propertyObject = undefined) {
    if (typeof proto !== 'object' && typeof proto !== 'function') {
        throw new TypeError('Object prototype may only be an Object or null.')
    if (propertyObject == null) {
        new TypeError('Cannot convert undefined or null to object')
    }
    function F() {}
    F.prototype = proto //关键 让obj的__proto__指向proto
    const obj = new F()
    if (propertyObject != undefined) {
        Object.defineProperties(obj, propertyObject)
    }
    if (proto === null) {
        // 创建一个没有原型对象的对象，Object.create(null)
        obj.__proto__ = null
    }
    return obj
}

```



##### 实现Object.assign

```js
Object.assign2 = function(target, ...source) {
    if (target == null) {
        throw new TypeError('Cannot convert undefined or null to object')
    }
    let ret = Object(target) 
    source.forEach(function(obj) {
        if (obj != null) {
            for (let key in obj) {
                if (obj.hasOwnProperty(key)) {
                    ret[key] = obj[key]
                }
            }
        }
    })
    return ret
}
```



##### 实现JSON.stringify  00

```js
function jsonStringify(data){
    let dataType = typeof data
    if(dataType != 'object'){
        let result = data
        if(Number.isNaN(data) || data === Infinity){
            result = "null"
        }else if(dataType === 'function' || dataType === 'symbol' || dataType === 'undefined'){
            return undefined
        }else if(dataType === 'string'){
            result = '"' + data + '"'
        }
        return String(result)
    }else if(dataType === 'object'){
        if(data === null){
            return null
        }else if(data.toJSON && typeof data.toJSON === 'function'){
            return jsonStringify(data.toJSON)
        }else if(data instanceof Array){
            let result = []
            data.forEach((item,index)=>{
                if(typeof item === 'undefined' || typeof item === 'function' || typeof item === 'symbol'){
                    result[index] = "null"
                }else{
                    result[index] = jsonStringify(item)
                }
            })
            result = '[' + result + ']'
            return result.replace(/'/g,'"')
        }else{
            // 普通对象
            let result = []
            Object.keys(data).forEach((item,index)=>{
                if(typeof item != 'symbol'){
                    if(typeof data[item] != 'symbol' && typeof data[item] != undefined && typeof data[item] !='function'){
                        result.push('"' + item +'"' + ':' + jsonStringify(data[item]))
                    }
                }
            })
            return ("{" + result + "}").replace(/'/g,'"')
        }
    }
}
```



##### 实现JSON.parse

```js
var json = '{"name":"小姐姐", "age":20}';
var obj = (new Function('return ' + json))();
```



##### 手写async await

```js
// 接收生成器作为参数，建议先移到后面，看下生成器中的代码
var myAsync = (generator) => {
  // 注意 iterator.next() 返回对象的 value 是 promiseAjax()，一个 promise
  const iterator = generator();

  // handle 函数控制 async 函数的 挂起-执行
  const handle = (iteratorResult) => {
    if (iteratorResult.done) return;

    const iteratorValue = iteratorResult.value;

    // 只考虑异步请求返回值是 promise 的情况
    if (iteratorValue instanceof Promise) {
      // 递归调用 handle，promise 兑现后再调用 iterator.next() 使生成器继续执行
      iteratorValue
        .then((result) => handle(iterator.next(result)))
        .catch((e) => iterator.throw(e));
    }
  };

  try {
    handle(iterator.next());
  } catch (e) {
    console.log(e);
  }
};

myAsync(function* () {
  try {
    const a = yield Promise.resolve(1);
    const b = yield Promise.resolve(a + 10);
    const c = yield Promise.resolve(b + 100);
    console.log(a, b, c); // 输出 1，11，111
  } catch (e) {
    console.log("出错了：", e);
  }
});
```



###### Generator原理

https://blog.csdn.net/weixin_43964148/article/details/107917507





##### Promise

```js
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';

class Promise {
    constructor(executor) {
        this.status = PENDING;
        this.value = undefined;
        this.reason = undefined;
        this.onResolvedCallbacks = [];
        this.onRejectedCallbacks = [];
        
        let resolve = (value) => {
            if (this.status === PENDING) {
                this.status = FULFILLED;
                this.value = value;
                this.onResolvedCallbacks.forEach((fn) => fn());
            }
        };
        
        let reject = (reason) => {
            if (this.status === PENDING) {
                this.status = REJECTED;
                this.reason = reason;
                this.onRejectedCallbacks.forEach((fn) => fn());
            }
        };
        
        try {
            executor(resolve, reject);
        } catch (error) {
            reject(error);
        }
    }
    
    then(onFulfilled, onRejected) {
        // 解决 onFufilled，onRejected 没有传值的问题
        onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
        // 因为错误的值要让后面访问到，所以这里也要抛出错误，不然会在之后 then 的 resolve 中捕获
        onRejected = typeof onRejected === "function" ? onRejected : (err) => {
            throw err;
        };
        // 每次调用 then 都返回一个新的 promise
        let promise2 = new Promise((resolve, reject) => {
            if (this.status === FULFILLED) {
                //Promise/A+ 2.2.4 --- setTimeout
                setTimeout(() => {
                    try {
                        let x = onFulfilled(this.value);
                        // x可能是一个proimise
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                }, 0);
            }
        
            if (this.status === REJECTED) {
                //Promise/A+ 2.2.3
                setTimeout(() => {
                    try {
                        let x = onRejected(this.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                }, 0);
            }
            
            if (this.status === PENDING) {
                this.onResolvedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onFulfilled(this.value);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch (e) {
                            reject(e);
                        }
                    }, 0);
                });
            
                this.onRejectedCallbacks.push(() => {
                    setTimeout(() => {
                        try {
                            let x = onRejected(this.reason);
                            resolvePromise(promise2, x, resolve, reject);
                        } catch (e) {
                            reject(e);
                        }
                    }, 0);
                });
            }
        });
        
        return promise2;
    }
}
const resolvePromise = (promise2, x, resolve, reject) => {
    // 自己等待自己完成是错误的实现，用一个类型错误，结束掉 promise  Promise/A+ 2.3.1
    if (promise2 === x) {
        return reject(
            new TypeError("Chaining cycle detected for promise #<Promise>"));
    }
    // Promise/A+ 2.3.3.3.3 只能调用一次
    let called;
    // 后续的条件要严格判断 保证代码能和别的库一起使用
    if ((typeof x === "object" && x != null) || typeof x === "function") {
        try {
            // 为了判断 resolve 过的就不用再 reject 了（比如 reject 和 resolve 同时调用的时候）  Promise/A+ 2.3.3.1
            let then = x.then;
            if (typeof then === "function") {
            // 不要写成 x.then，直接 then.call 就可以了 因为 x.then 会再次取值，Object.defineProperty  Promise/A+ 2.3.3.3
                then.call(
                    x, (y) => {
                        // 根据 promise 的状态决定是成功还是失败
                        if (called) return;
                        called = true;
                        // 递归解析的过程（因为可能 promise 中还有 promise） Promise/A+ 2.3.3.3.1
                        resolvePromise(promise2, y, resolve, reject);
                    }, (r) => {
                        // 只要失败就失败 Promise/A+ 2.3.3.3.2
                        if (called) return;
                        called = true;
                        reject(r);
                    });
            } else {
                // 如果 x.then 是个普通值就直接返回 resolve 作为结果  Promise/A+ 2.3.3.4
                resolve(x);
            }
        } catch (e) {
            // Promise/A+ 2.3.3.2
            if (called) return;
            called = true;
            reject(e);
        }
    } else {
        // 如果 x 是个普通值就直接返回 resolve 作为结果  Promise/A+ 2.3.4
        resolve(x);
    }
};

```



`catch`方法还有一个作用，就是在执行`resolve`回调函数时，如果出现错误，抛出异常，不会停止运行，而是进入`catch`方法中

###### Promise.resolve

```js
Promise.resolve = function(value){
    if(value instanceof Promise){
        return value
    }else{
        return new Promise((resolve)=>{resolve(value)})
    }
}
```



###### Promise.reject

```js
Promise.reject = function (reason){
        return new Promise((resolve,reject)=>{reject(reason)})
}
```



###### Promise.all

```js
Promise.all = function(promiseArr) {
    let index = 0, result = []
    return new Promise((resolve, reject) => {
        promiseArr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                index++
                result[i] = val
                if (index === promiseArr.length) {
                    resolve(result)
                }
            }, err => {
                reject(err)
            })
        })
    })
}

```



###### Promise.race

```js
Promise.race = function(promiseArr) {
    return new Promise((resolve, reject) => {
        promiseArr.forEach(p => {
            Promise.resolve(p).then(val => {
                resolve(val)
            }, err => {
                rejecte(err)
            })
        })
    })
}

```



###### Promise.any

```js
Promise.any = function(promiseArr) {
    let index = 0
    return new Promise((resolve, reject) => {
        if (promiseArr.length === 0) return 
        promiseArr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                resolve(val)
                
            }, err => {
                index++
                if (index === promiseArr.length) {
                  reject(new AggregateError('All promises were rejected'))
                }
            })
        })
    })
}

```



###### Promise.allSettled

```js
Promise.allSettled = function(promiseArr) {
    let result = []
    return new Promise((resolve, reject) => {
        promiseArr.forEach((p, i) => {
            Promise.resolve(p).then(val => {
                result.push({
                    status: 'fulfilled',
                    value: val
                })
                if (result.length === promiseArr.length) {
                    resolve(result) 
                }
            }, err => {
                result.push({
                    status: 'rejected',
                    reason: err
                })
                if (result.length === promiseArr.length) {
                    resolve(result) 
                }
            })
        })  
    })   
}

```





##### Promise实战



### 并发限制request池

```js
function request(url){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve()
            console.log(`request ${url}`);
        },1000)
    })
}

function addTask(url){
    const task = request(url)
    pool.push(task)
    task.then(()=>{
        pool.splice(pool.indexOf(task),1)
        let newUrl = urls.shift()
        if(newUrl!=undefined){
            addTask(newUrl)
        }
    })
}

let urls =  ['bytedance.com','tencent.com','alibaba.com','microsoft.com','apple.com','hulu.com','amazon.com'] // 请求地址
let pool = []//并发池
let max = 3 //最大并发量
for(let i=0;i<max;i++){
    addTask(urls.shift())
}
```





###### 带并发限制的异步调度器Scheduler

```js
class Scheduler{
    constructor(limit){
        this.cache = []
        this.tasks = []
        this.limit = limit
    }
    add(promiseCreator){
        return new Promise((resolve,reject)=>{
            promiseCreator.resolve = resolve
            if(this.tasks.length<this.limit){
                this.tasks.push(promiseCreator)
                this.runTask(promiseCreator)
            }else{
                this.cache.push(promiseCreator)
            }
        })
    }

    runTask(promiseCreator){
        promiseCreator().then(()=>{
            promiseCreator.resolve()
            this.tasks.splice(this.tasks.indexOf(promiseCreator),1)
            if(this.cache.length>0){
                let newTask = this.cache.shift()
                this.tasks.push(newTask)
                this.runTask(newTask)
            }
        })
    }
}

let scheduler = new Scheduler(2)
let timeFunc = (time)=>{
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            resolve()
        },time)
    })
}
addTask = (time,order) =>{
    let ret = scheduler.add(()=>timeFunc(time))
    ret.then(()=>{
        console.log(order);
    })
}

addTask(1000,'1')
addTask(500,'2')
addTask(300,'3')
addTask(400,'4')
```



###### LazyMan

```js
class LazyManClass {
  constructor(name) {
    this.name = name
    this.queue = []
    console.log(`Hi I am ${name}`)
    setTimeout(() => {
      this.next()
    },0)
  }

  sleepFirst(time) {
    const fn = () => {
      setTimeout(() => {
        console.log(`等待了${time}秒...`)
        this.next()
      }, time)
    }
    this.queue.unshift(fn)
    return this
  }

  sleep(time) {
    const fn = () => {
      setTimeout(() => {
        console.log(`等待了${time}秒...`)
        this.next()
      },time)
    }
    this.queue.push(fn)
    return this
  }

  eat(food) {
    const fn = () => {
      console.log(`I am eating ${food}`)
      this.next()
    }
    this.queue.push(fn)
    return this
  }

  next() {
    const fn = this.queue.shift()
    fn && fn()
  }
}

function LazyMan(name) {
  return new LazyManClass(name)
}

new LazyMan('Hank').sleep(10000).eat('dinner').next()
// Hi! This is Hank!
//等待10秒..
// Wake up after 10
// Eat dinner~

new LazyMan('Hank').eat('dinner').eat('supper')
// Hi This is Hank!
// Eat dinner~
// Eat supper~

new LazyMan('Hank').sleepFirst(5).eat('supper')
//等待5秒
// Wake up after 5
// Hi This is Hank!
// Eat supper
```

















### 洗牌算法

```js
//将随机的一个数放到最后 直到全部放完
function Shuffle(arr){
    let len = arr.length
    while(len>0){
        let rand = Math.floor(Math.random()*len)
        len--
        [arr[rand],arr[len]] = [arr[len],arr[rand]]
    }
    return arr
}
```





### 虚拟列表（固定长度）

```react
import React from 'react'
import './App.css'

export default class App extends React.Component {
  state={
    listData:[],
    startIndex:0,
    endIndex:10,
    itemHeight:100,
    visibleData:[],
    scrollTop:0,
    visibleCount: Math.ceil(window.screen.height/100),
    startOffset:0
  }
  componentDidMount(){
    const {startIndex,visibleCount} = this.state
    let listData = this.getData()
    let endIndex = startIndex + visibleCount
    let visibleData = listData.slice(startIndex,endIndex)
    this.setState({
      listData,
      endIndex,
      visibleData
    })
  }

  getData = ()=>{
    let data = []
    for(let i=0;i<10000;i++){
      data.push(i)
    }
    return data
  }

  onScrollEvent=()=>{
    const {itemHeight,visibleCount,listData} = this.state
    const scrollTop = this.containerRef.current.scrollTop
    let startIndex = Math.floor(scrollTop/itemHeight)
    let endIndex = startIndex + visibleCount
    let visibleData = listData.slice(startIndex,endIndex)
    let startOffset = scrollTop - (scrollTop % itemHeight)
    this.listRef.current.style = `transform: translate3d(0,${startOffset}px,0)`
    this.setState({
      startIndex,
      endIndex,
      startOffset,
      visibleData
    })
  }

  containerRef = React.createRef()
  listRef = React.createRef()
  render() {
    const {visibleData,itemHeight} = this.state
    return (
        <div className='infinite-list-container' ref={this.containerRef} onScroll={this.onScrollEvent}>
          <div className='infinite-list-phantom' style={{height:'1000000px'}}></div>
          <div className='infinite-list' ref={this.listRef}>
            {visibleData.map((item)=>
              (
              <div 
              className='infinite-list-item'
              style={{height:itemHeight+'px',lineHeight:itemHeight+'px'}}
              key={item}
              >
              {item}
              </div>
            )
            )}
          </div>
        </div>
    )
  }
}



//App.css
#root{
  height:100%;
}
.infinite-list-container {
  height: 100%;
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}

.infinite-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.infinite-list {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
  text-align: center;
}

.infinite-list-item {
  padding: 10px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
}

//index.css
html{
  height: 100%;
}
body{
  height: 100%;
  margin:0;
}
```



### 虚拟列表 （不定长度）

```react
import './App.css';
import React, { Component } from 'react'
import { faker } from '@faker-js/faker';
class App extends React.Component {
  state ={
    estimatedItemHeight:50,
    positions:[],
    listData:[],
    startIndex: 0,
    visibleCount:Math.ceil(window.screen.height / 50),
    endIndex: 0,
    visibleData:[],
    mylock: false,
    height:0
  }

  componentDidMount(){
    const {startIndex,visibleCount,estimatedItemHeight} = this.state
    const listData = this.getFakeData()
    const endIndex = startIndex + visibleCount
    const visibleData = listData.slice(startIndex,endIndex)
    const positions = listData.map((d,index)=>({
      index,
      height:estimatedItemHeight,
      top: index*estimatedItemHeight,
      bottom: (index+1) * estimatedItemHeight
    }))
    this.setState({
      listData,
      endIndex,
      visibleData,
      positions
    })
  }

  componentDidUpdate(_,prevState){
    console.log(this.state,'===',prevState);
    if(prevState.mylock == true){
      return
    }
    this.updateItemsSize()
    let height= this.state.positions[this.state.positions.length-1].bottom
    // console.log(height);
    this.phantomRef.current.style.height = height + 'px';
    this.setStartOffset()
    this.setState({
      mylock:true,
      height
    })
  }


  updateItemsSize = ()=>{
    let nodes = Array.from(this.contentRef.current.children)
    console.log(this.contentRef.current.children);
    let {positions} = this.state
    nodes.forEach((node)=>{
      let rect = node.getBoundingClientRect()
      let height = rect.height
      let index = +node.id
      let oldHeight = positions[index].height
      let dValue = oldHeight - height
      if(dValue){
        positions[index].bottom = positions[index].bottom - dValue
        positions[index].height = height
        for(let i=index+1;i<positions.length;i++){
          positions[i].top = positions[i-1].bottom
          positions[i].bottom = positions[i].bottom - dValue
        }
      }
    })
    this.setState({
      positions
    })
  }


  contentRef = React.createRef()
  phantomRef = React.createRef()
  itemsRef = React.createRef()
  listRef = React.createRef()

  getFakeData = () => {
    let d = []
    for(let id =0;id<10000;id++){
      d.push({
        id,
        value:faker.lorem.sentences()
      })
    }
    return d
  }

  getStartIndex = (scrollTop)=>{
    return this.binarySearch(this.state.positions,scrollTop)
  } 

  binarySearch = (positions,scrollTop)=>{
    let start = 0
    let end = positions.length-1
    let tempIndex = null
    while(start<end){
      let mid = start + Math.floor((end-start)/2)
      if(positions[mid].bottom ==scrollTop){
        return mid+1
      }else if(positions[mid].bottom<scrollTop){
        start = mid+1
      }else if(positions[mid].bottom>scrollTop){
        if(tempIndex == null || tempIndex > mid){
          tempIndex = mid 
        }
        end = mid -1
      }
    }
    return tempIndex
  }

  setStartOffset=()=>{
    let startOffset =this.state.startIndex >= 1 ? this.state.positions[this.state.startIndex - 1].bottom : 0;
    this.contentRef.current.style.transform = `translate3d(0,${startOffset}px,0)`;
  }

  onScrollEvent= ()=>{
    const {visibleCount,listData} = this.state 
    const scrollTop = this.listRef.current.scrollTop
    const start = this.getStartIndex(scrollTop)
    const end = start + visibleCount
    const visibleData = listData.slice(start,end)
    this.setStartOffset()
    this.setState({
      startIndex:start,
      endIndex:end,
      visibleData,
      mylock:false
    }) 
  }
  
  render() {
    const {visibleData,height} = this.state
    return (
      <div className="infinite-list-container" onScroll={this.onScrollEvent} ref={this.listRef} style={{height: '100%'}}>
          <div className="infinite-list-phantom" ref={this.phantomRef}></div>
          <div className="infinite-list" ref={this.contentRef}>
              {visibleData.map((item,index)=>(
                  <div 
                  ref={this.itemsRef[index]}
                  className="infinite-list-item"
                  key={item.id}
                  id = {item.id}>
                  {item.id}
                  {item.value}
                  </div>
                ))}
          </div>
      </div>
    )
  }
}

export default App;

//app.css

#root{
  height:100%;
}
.infinite-list-container {
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}
.infinite-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}
.infinite-list {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
  text-align: center;
}
.infinite-list-item {
  padding: 10px;
  color: #555;
  box-sizing: border-box;
  border-bottom: 1px solid #999;
}


//index.css
html{
  height: 100%;
}
body{
  height: 100%;
  margin:0;
}
#app{
  height:100%;
}
```









![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/1/6/16f7b3db4a239442~tplv-t2oaga2asx-zoom-in-crop-mark:3024:0:0:0.awebp)



### 排序算法

[动图展示10大排序算法](https://www.cnblogs.com/onepixel/articles/7674659.html)

##### 冒泡排序

```js
function bubbleSort(arr){
    for(let i=0;i<arr.length;i++){
        let mark = true //优化  
        for(let j=0;j<arr.length-i-1;j++){
            if(arr[j]>arr[j+1]){
                [arr[j],arr[j+1]] = [arr[j+1],arr[j]]
                mark = false
            }
        }
        if(mark){
            return arr //如果直接是排好了序的话 直接返回arr
        }
    }
    return arr
}
```



##### 选择排序

```js
function selectSort(arr){
    let len = arr.length
    let index
    for(let i=0;i<len;i++){
        index = i
        for(let j=i+1;j<len;j++){
            //找最小的
            if(arr[j]<arr[index]){
                index = j
            }
        }
        [arr[i],arr[index]] = [arr[index],arr[i]]
    }
    return arr
}
```



##### 插入排序

```js
function insertSort(arr){
    let len=arr.length
    let preIndex,current
    for(let i=1;i<len;i++){
        preIndex = i-1
        current = arr[i]	
        while(preIndex>=0 && arr[preIndex]>current){
            arr[preIndex+1] = arr[preIndex]
            preIndex--
        }
        arr[preIndex+1] = current
    }
    return arr
}

//加入二分法优化
function binsertSort(arr){
    let len = arr.length
    let j 
    for(let i=1;i<len;i++){
        let current = arr[i]
        if(arr[i-1]>arr[i]){
            let low = 0
            let high = i-1
            while(low<=high){
                let mid = low + Math.floor((high-low)/2)
                if(arr[mid]>current){
                    high = mid-1
                }else{
                    low = mid+1
                }
            }
            for(j=i;j>low;j--){
                arr[j] = arr[j-1]
            }
            arr[j] = current
        }
    }
    return arr
}
```



##### 希尔排序

```JS
function shellSort(arr){
    let len = arr.length
    let gap = Math.floor(len/2)
    while(gap){
        for(let i=gap;i<len;i++){
            for(let j=i-gap;j>=0;j-=gap){
                if(arr[j]>arr[j+gap]){
                    [arr[j],arr[j+gap]] = [arr[j+gap],arr[j]]
                }else{
                    break //因为前面都排好序列了 所以可以直接Break退出
                }
            }
        }
        gap = Math.floor(gap/2)
    }
    return arr
}
```



##### 归并排序

```js
//自顶向下递归法
function mergeSort(arr){
    let len = arr.length
    if(len<2){
        return arr
    }
    let mid = Math.floor(len/2)
    let left = arr.slice(0,mid)
    let right =arr.slice(mid)
    return merge(mergeSort(left),mergeSort(right))
}
function merge(left,right){
    let result = []
    while(left.length>0&&right.length>0){
        if(left[0]<right[0]){
            result.push(left.shift())
        }else{
            result.push(right.shift())
        }
    }
    while(left.length){
        result.push(left.shift())
    }
    while(right.length){
        result.push(right.shift())
    }
    return result
}
```



##### 快速排序

```js
function quickSort(arr,begin,end){
    if(begin>end){
        return arr
    }
    let i=begin,j=end
    let temp = arr[begin]
    while(i!=j){
        //关键点 先从右边开始！！！ 必须 因为这样才能保证交换的时候i位置上的数一定是小于等于基数的
        while(arr[j]>=temp&&i<j){
            j--
        }
        while(arr[i]<=temp&&i<j){
            i++
        }
        if(i<j){
            [arr[i],arr[j]] = [arr[j],arr[i]]
        }
    }
    //找到了i位置的数 和begin的数进行交换
    [arr[begin],arr[i]] = [arr[i],temp]
    //因为i左小于i 右大于i  所以下一步从begin到i-1 再下一步i+1到end
    quickSort(arr,begin,i-1)
    quickSort(arr,i+1,end)
    return arr
}

function quickSort(arr,begin,end){
    if(begin>end) return arr   //复习重点1 要注意终止条件
    let item = arr[begin]
    let left = begin 
    let right = end
    while(left!=right){ //复习重点3 这里left!=right
        while(arr[right]>=item&&left<right) right-- //复习重点4 要从右边开始
        arr[left] = arr[right]
        while(arr[left]<=item&&left<right) left++
        arr[right] = arr[left]
    }
    arr[left] = item
    quickSort(arr,begin,left-1) //复习重点2 要注意这里left-1 下面left+1
    quickSort(arr,left+1,end)
    return arr
}
```



##### 计数排序

```js
function countSort(arr){
    let result = []
    let min =Math.min(...arr)
    let max =Math.max(...arr)
    for(let i=0;i<arr.length;i++){
        let temp = arr[i]
        result[temp] = result[temp]+1||1
    }
    let index=0
    for(let i=min;i<=max;i++){
        while(result[i]>0){
            arr[index++] = i
            result[i]--
        }
    }
    return arr
}
```



##### 基数排序

```js
function radixSort(nums){
    function getDigits(n){
        let count = 0
        while(n){
            count++
            n = Math.floor(n/10)
        }
        return count
    }
    let max = Math.max(...nums)
    let maxDigits = getDigits(max)
    let arr = new Array(10).fill(0).map(()=>new Array())
    for(let i=0;i<nums.length;i++){
        nums[i] = (nums[i]+'').padStart(maxDigits,0)
        let temp = nums[i][nums[i].length-1]
        arr[temp].push(nums[i])
    }
    nums = [].concat(...arr)
    arr = new Array(10).fill(0).map(()=>new Array())
    for(let i=maxDigits-2;i>=0;i--){
        for(let j=0;j<nums.length;j++){
            let temp = nums[j][i]
            arr[temp].push(nums[j])
        }
        nums = [].concat(...arr)
        arr = new Array(10).fill(0).map(()=>new Array())
    }
    nums = nums.map((item)=>parseInt(item))
    return nums
}
```















##### 堆排序

```js
function heapSort(arr){
    buildHeap(arr)
    //终止条件i>0
    for(let i=arr.length-1;i>0;i--){
        //调整最大值到底部
        [arr[0],arr[i]] = [arr[i],arr[0]]
        adjustHeap(arr,0,i)
    }
    return arr
}

function buildHeap(arr){
    let len = arr.length
    let start = Math.floor(len/2) - 1 //从第一个非叶子节点开始
    for(let i=start;i>=0;i--){
        adjustHeap(arr,i,len)
    }
}

function adjustHeap(arr,i,len){
    while(true){
        let left = i*2+1
        let right = i*2+2
        let largest = i
        if(left<len&&arr[left]>arr[largest]){
            largest = left
        }
        if(right<len&&arr[right]>arr[largest]){
            largest = right
        }
        if(largest!=i){
            //如果不平衡就调整 然后递归
            [arr[largest],arr[i]] = [arr[i],arr[largest]]
            i=largest
        }else{
            break
        }
    }
}
```





###### 堆排实战

[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

参考文献：https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution/xie-gei-qian-duan-tong-xue-de-ti-jie-yi-kt5p2/

```js
 // 整个流程就是上浮下沉
var findKthLargest = function(nums, k) {
   let heapSize=nums.length
    buildMaxHeap(nums,heapSize) // 构建好了一个大顶堆
    // 进行下沉 大顶堆是最大元素下沉到末尾
    for(let i=nums.length-1;i>=nums.length-k+1;i--){
        swap(nums,0,i)
        --heapSize // 下沉后的元素不参与到大顶堆的调整
        // 重新调整大顶堆
         maxHeapify(nums, 0, heapSize);
    }
    return nums[0]
   // 自下而上构建一颗大顶堆
   function buildMaxHeap(nums,heapSize){
     for(let i=Math.floor(heapSize/2)-1;i>=0;i--){
        maxHeapify(nums,i,heapSize)
     }
   }
   // 从左向右，自上而下的调整节点
   function maxHeapify(nums,i,heapSize){
       let l=i*2+1
       let r=i*2+2
       let largest=i
       if(l < heapSize && nums[l] > nums[largest]){
           largest=l
       }
       if(r < heapSize && nums[r] > nums[largest]){
           largest=r
       }
       if(largest!==i){
           swap(nums,i,largest) // 进行节点调整
           // 继续调整下面的非叶子节点
           maxHeapify(nums,largest,heapSize)
       }
   }
   function swap(a,  i,  j){
        let temp = a[i];
        a[i] = a[j];
        a[j] = temp;
   }
};
```

