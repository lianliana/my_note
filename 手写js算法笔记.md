### 函数柯里化

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

### bind

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

