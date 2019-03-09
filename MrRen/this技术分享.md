# this 分享

### 什么是this？

​	当一个函数调用时，会创建一个活动记录也就是执行上下文。调用记录包含了，调用栈、传入的参数、函数的调用方法。this 就是这个记录上的一个属性，在函数运行时动态绑定，它的上下文取决于函数调用时的各种条件。和函数申明的位置无关。（取决于调用位置）

this 不是指向函数自身，也不是指向函数的词法作用域。

### 使用调试工具查看函数调用栈

​	在函数中第一句使用 debugger，查看调用栈，第二行为当前函数的调用环境。

```javascript
function fa(){
    debugger;
    console.log("fa");
    fb();
}
function fb(){
    debugger;
    console.log("fb");
    fc();
}

function fc(){
    debugger;
    console.log("fc")
}

```



### this绑定规则

 * 第一种：默认绑定

   默认绑定旨在，函数独立调用时默认绑定this。浏览器环境下为window（注意非严格模式），严格模式下为undefined。优先级是最低的一种绑定方式。

* 第二种：隐式绑定

  规则：调用位置是否有上下文对象，（函数调用时被某些对象拥有）

  ```javascript
  function foo (){
      debugger;
      console.log(this.pro);
  }
  var pro =" global "；
  var obj = {
      pro:"obj",
      foo:foo
  }
  obj.foo()
  
  ```

  ​	隐式丢失：当一个对象上的函数被一个无任何修饰的变量保存引用后，隐式绑定规则失效（进而只能使用默认的绑定规则）

  ​	

  ```javascript
  var obj = {
      foo(){
          console.log(this.pro);
      },
      pro:"obj"
  }
  var pro = "global";
  var foo = obj.foo;//function（）{console.log(this.pro)}
  foo()
  
  ```

  ​	注：通常这种情况发生最多的是回调函数失去 this 。

* 显示绑定

  在开发中我们通常最喜欢使用这样的方式来，强迫一个对象去贡献一些数据。call/apply 的使用，他们都是显示的将函数绑定到一个对象上来强制改变 this 的指向问题。不同的是它们的参数传递方式的不同。

  

  ```javascript
  function foo(){
      console.log(this.pro);
  }
  var obj = {
      pro:"obj"
  }
  foo.call(obj);
  
  ```

  ​	显然这也不饿能解决，回调函数失去（活动太改变）this。

  *  硬绑定

    ​	借助显示绑定的原理，包装一层函数来实现 。

     

    ```javascript
    function foo(){
        console.log(this.pro);
    }
    var obj = {
        pro:"obj"
    }
    var fa = function (obj){
        foo.call(obj);
    }
    
    fa.call(window,obj);//即使这样也能不改变 foo 硬绑定上去的 this 对象
    
    ```

  * 模仿一下  Function.prototype.bind 函数

    ​	

    ```javascript
    
    function bind(fn,obj){
        return function (){
            return fn.apply(obj,aguments);
        }
    }
    
    ```

    

​            

* new 绑定

  ​	构造调用函数时，会有这么一个影响 this的过程，函数首先会在内部创建一个对象=》然后执行原型连接=》将这个对象绑定到函数调用的this 上=》最后在没有返回对象的情况下返回这个对象。

  ```javascript
  function Foo(a){
  	this.a =a;	
  }
  let obj = new Foo(22);
  
  console.log(obj.a)// 22
  
  ```

###  绑定的优先级 

​	默认绑定是优先级最低的前面有说到, 这儿我们就不讨论它了。

 * 隐式绑定和显示绑定的比较

```javascript
function foo(){
    console.log(this.pro);
}

var obj1 = {
    pro:"obj1",
    foo:foo
}
var obj2 = {
    pro:"obj2",
    foo:foo
}

obj1.foo();// obj1
obj2.foo();// obj2

obj1.foo.call(obj2);// obj2

```

​	显然显示绑定优先级更高，因此我们呢考虑 this 绑定的时候应该先考虑显示绑定。

* 隐式绑定与 new 的比较

  ​	

  ```javascript
  function foo(a){
      this.a = a;
  }
  var obj={
      foo:foo
  }
  
  obj.foo(2);
  console.log(obj.a);//2
  
  var obj2 = new obj.foo(3);
  console.log(obj2.a);//3
  
  ```

  ​	结论：new 的优先级比 隐式高要。

*  显示绑定与 new 的比较

  ​	因为call /apply 不能同 new 同时使用，因此我们借用硬绑定（显示绑定的一种）简单直接使用bind函数来实现。

  ```javascript
  function foo(a){
      this.a = a; 
  }
  var obj = {};
  var bar = foo.bind(obj);
  bar(1);
  console.log(obj.a);//1
  
  var obj2 = new bar(2);
  console.log(obj2.a);//2
  
  
  ```

  ​	呀呵！new 的 优先级比显示绑定更高些。因此得出一些结论。

  ​	优先级：new > 显示绑定 > 隐式绑定 > 默认绑定

  

### 特殊的绑定

​	在实现显示绑定的时候如果传入，null、undefined 作为 call /apply 的第一个参数，那么实际上将使用默认绑定。但是为什么会有这个需求呢？因为柯里化的一种需要使用 bind 来预先传入一些参数（柯里化实际上就是将参数减少）。

```javascript
foo.bind(null,name,age);
```

但是会带来默认的绑定，这就可能修改window（全局）的一些属性。有可能这是没有预期的，带来的 bug 很难追踪到。为此需要新建一个 ‘ 假装是空的对象’（不需要Object.prototype 属性），Object.create(null) 来创建一个比{}更空的对象。

![1552034074268](C:\Users\MrRen\AppData\Roaming\Typora\typora-user-images\1552034074268.png)



### 绑定误区

​	眼睛有点闹腾直接看代码吧！

​		

```javascript
function foo(){
	console.log(this.a)
}
var a = 111;
var obj = {a:222,foo:foo};
var obj1 = {a:333}
(obj1.foo = obj.foo)()；//111

```



### 很软的一种绑定

​	soft bind ，当使用硬绑定后，隐式绑定和显示绑定就无法修改 this 了。因此硬绑定降低了函数的灵活性，与JavaScript 如此灵活（简直想哭）格格不入。如果给默认绑定指定一个除全局对象和undefined外的对象，这就达到了软绑定的需求。（优先级低，但不是全局对象做this）。

​	

```javascript
if(!Function.prototype.softBind){
    Function.prototype.softBind = function (obj){
        var fn = this;
        var curried = [].slice.call(arguments,1);
        var bound = function (){
            return fn.apply((!this || this === ( window || global))? obj:this
                            ,curried.cancat.apply(curried,arguments)
                           );
        };
        bound.prototype = Object.create(fn.prototype);
        return bound;
    }
}
```

### 箭头函数说在最后

​	箭头函数不受四种绑定规则的约定，它将外部作用域作为this。而且，箭头函数的 this 不被改变（使用优先级最高的new也不行）因为它不使用 new 关键字。改变了 this的机制。具体再举个列：

​	

```javascript
function foo (){
    setTimeout(()=>{
        console.log(this.a);
    },100)
}
var obj = {a:123};
obj.foo = foo;
var obj1 = {a:456};
obj.foo.bind(obj1);
obj.foo();//123

```

