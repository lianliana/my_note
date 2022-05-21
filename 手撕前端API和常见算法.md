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



##### 数据类型判断

```js
function myTypeOf(obj){
        return Object.prototype.toString.call(obj).slice(8,-1).toLowerCase()
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
// - Dog.prototype.constructor =Dog
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







##### 函数柯里化

```js
function sum(...args){
        return args.reduce((a,b)=>{
            return a+b
        })
    }

  function currying(fn){
      //闭包存储参数
        const args=[]
      //返回柯里化后函数
        return function result(...res){
            //最后一次调用返回结果
            if(res.length===0){
                return fn(...args)
            }else{
                //否则push入参数并且返回result函数，以便于链式调用
                args.push(...res)
                return result
            }
        }
    }
```





##### bind

```js	

Function.prototype.my_bind = function(context){
    var args = Array.prototype.slice.call(arguments, 1);
    //args [7, 8]
    var self = this;
    return function(){
        var innerArgs = Array.prototype.slice.call(arguments);
        //innerArgs [9]
        var finalArgs = args.concat(innerArgs);
        //finalArgs [7, 8, 9]
        return self.apply(context, finalArgs);
    }

//测试
function a(m, n, o){
    return this.name + ' ' + m + ' ' + n + ' ' + o;
}
 
var b = {name : 'kong'};
 
a.my_bind(b, 7, 8)(9);        //kong 7 8 9
```