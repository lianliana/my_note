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







### LazyMan

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
```



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
var events = (function() {
  var topics = {};

  return {
    // 注册监听函数
    subscribe: function(topic, handler) {
      if (!topics.hasOwnProperty(topic)) {
        topics[topic] = [];
      }
      topics[topic].push(handler);
    },

    // 发布事件，触发观察者回调事件
    publish: function(topic, info) {
      if (topics.hasOwnProperty(topic)) {
        topics[topic].forEach(function(handler) {
          handler(info);
        });
      }
    },

    // 移除主题的一个观察者的回调事件
    remove: function(topic, handler) {
      if (!topics.hasOwnProperty(topic)) return;

      var handlerIndex = -1;
      topics[topic].forEach(function(item, index) {
        if (item === handler) {
          handlerIndex = index;
        }
      });

      if (handlerIndex >= 0) {
        topics[topic].splice(handlerIndex, 1);
      }
    },

    // 移除主题的所有观察者的回调事件
    removeAll: function(topic) {
      if (topics.hasOwnProperty(topic)) {
        topics[topic] = [];
      }
    }
  };
})();
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
        let context=this
        let args=arguments
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

```js
function throttle(func, wait){
    let context,args
    let prev = 0
    return function(){
        context = this
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



##### 实现JSON.stringify

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
    if(begin>end) return arr
    let item = arr[begin]
    let left = begin 
    let right = end
    while(left!=right){
        while(arr[right]>=item&&left<right) right--
        arr[left] = arr[right]
        while(arr[left]<=item&&left<right) left++
        arr[right] = arr[left]
    }
    arr[left] = item
    quickSort(arr,begin,left-1)
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
    for(let i=maxDigits-2;i>=0;i--){
        for(let j=0;j<=9;j++){
            let temp = arr[j]
            let len = temp.length
            while(len--){
                let str = temp.shift()
                arr[str[i]].push(str)
            }
        }
    }
    let res = [].concat(arr)
    res.forEach((val,index)=>{
        nums[index] = +res[index]
    })
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

