---
title: this机制的学习思考
date: 2018-02-23 15:30:03
tags: [JavaScript,this,ES6]
---

> this在不同场景下的指向规则以及ES6中新增的箭头函数中this的指向问题

<!--more-->

### 关于this
this提供了一种更优雅的方式来隐式"传递"一个对象引用,因此可以将API设计得更加简洁并且易于复用.

this常见的误解:

1. 指向自身
2. 指向函数的作用域
    1. this在任何情况下都不指向函数的词法作用域。在JavaScript内部，作用域确实和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过JavaScript代码访问，它存在于JavaScript引擎内部。
    

this的机制:

1. this是在运行时进行绑定的,并不是在编写时绑定,它的上下文取决于函数调用时的各种条件.this的绑定和函数声明的位置没有任何关系,只取决于函数的调用方式.
2. 当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this就是**记录的其中一个属性**，会在函数**执行的过程**中用到。



通过调用栈可以查看当前函数体内this的指向:
![](https://raw.githubusercontent.com/wkd2015/Pic/master/blog/this_1.png)

-------

### this解析

#### 调用位置:函数在代码中被调用的位置(而不是声明位置)
1. 调用栈:为了到达当前执行位置所调用的所有函数.
``` bash
function baz() {
    // 当前调用栈是：baz
    // 因此，当前调用位置是全局作用域
    console.log( "baz" );
    bar(); // <-- bar的调用位置
}
function bar() {
    // 当前调用栈是baz -> bar
    // 因此，当前调用位置在baz中
    console.log( "bar" );
    foo(); // <-- foo的调用位置
}
function foo() {debugger;
    // 当前调用栈是baz -> bar -> foo
    // 因此当前调用位置在bar中
       console.log( "foo" );
}
baz(); // <-- baz的调用位置
```
    
### this的五种绑定规则

#### 1. 默认绑定
直接使用不带任何修饰的函数引用进行调用的,会进行默认绑定。
```
function foo() {
    console.log(this.a);
}
var a = 2;
foo();  // 2
```
#### 2. 隐式绑定
调用位置是否有上下文对象,或者说是否被某个对象拥有或包含。
``` 
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
obj.foo(); // 2 
```
需要注意的是无论是直接在obj中定义还是先定义再添加为引用属性，这个函数严格来说都**不属于obj对象**。 然而，调用位置会使用obj上下文来引用函数，因此你可以说函数被调用时obj对象“拥有”或者“包含”它。  
当foo()被调用时，它的落脚点确实指向obj对象。当函数引用有上下文对象时，隐式绑定规则会把函数调用中的this绑定到这个上下文对象。因为调用foo()时this被绑定到obj，因此this.a和obj.a是一样的。 
##### **隐式丢失问题**
```
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var bar = obj.foo; // 函数别名！
var a = "oops, global"; // a是全局对象的属性
bar(); // "oops, global"
```
_虽然bar是obj.foo的引用,但实际上其引用的是foo本身,因此此时的bar是不带修饰的函数调用,因此应用默认绑定._
实际情况里比较常见的是回调函数的this丢失问题:
```
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var a = "oops, global"; // a是全局对象的属性
setTimeout( obj.foo, 100 ); // "oops, global"
```
_回调函数失去this值绑定也是一样的原因,func(fn)中fn其实是赋值给了函数的参数_
在上面这段代码里obj.foo在传入setTimeout时其实是做了一个隐式的赋值操作，因此，在setTimeout函数体内调用时，其实调用的参数直接引用了function foo(){}，导致this绑定到全局对象上。

#### 3. 显式绑定
像隐式绑定中的例子里，如果不想将函数包含在对象体内的属性上，那么可以利用显式绑定方法(call、apply),它们会把这个对象绑定到this，接着在调用函数时指定这个this。
```
function foo() { 
    console.log( this.a );
}
var obj = { 
    a:2
};
foo.call( obj ); // 2 
```
利用显式绑定的一个变种方法可以解决隐式丢失的问题，即创建一个包裹函数，传入所有的参数并返回接收到的所有值：
```
function foo(something) { 
    console.log( this.a, something ); 
    return this.a + something;
}
var obj = { 
    a:2
};
var bar = function() {
    return foo.apply( obj, arguments );
};
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
通过创建函数bar()，并在它的内部手动调用foo.call(obj)，从而强制把foo的this绑定到了obj。无论之后如何调用函数bar，它总会手动在obj上调用foo。这种绑定是一种显式的强制绑定。
在ES5中也提供了内置的这种绑定方法:Function.prototype.bind，bind(..)会返回一个硬编码的新函数，它会把参数设置为this的上下文并调用原始函数。

> bind()与call()和apply()这两种方法不同之处在于其处理后函数不会执行，而是返回了一个改变了上下文的函数副本，而后两者则会直接执行函数。

#### 4. new绑定
与其他语言中不同，在JavaScript中，构造函数只是一些使用new操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被new操作符调用的普通函数而已。  
包括内置对象函数在内的所有函数都可以用new来调用，这种函数调用被称为构造函数调用。这里有一个重要但是非常细微的区别：**实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”**。   

new操作符进行构造函数调用时的操作流程:
    1. 创建一个全新的对象
    2. 这个新对象会被执行\[\[原型]]连接
    3. 这个新对象会绑定到函数调用的this
    4. 如果函数返回没有其它对象,那么new表达式中的函数调用会自动返回这个新对象.
      
```
function foo(a) { 
    this.a = a;
} 
var bar = new foo(2);
console.log( bar.a ); // 2
```
使用new来调用foo(..)时，我们会构造一个新对象并把它绑定到foo(..)调用中的this上.

#### 利用优先级规则判断非箭头函数的this指向
1. 函数是否在new中调用（new绑定）？如果是的话this绑定的是新创建的对象。
2. 函数是否通过call、apply（显式绑定）或者硬绑定调用？如果是的话，this绑定的是指定的对象。
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。

#### 5. 箭头函数绑定
箭头函数是ES6新增的一种函数表达式，不是使用function关键字来定义，也不适用上述四条判定规则，而是**依据外层作用域来决定this**。  
1. 箭头函数可以让this绑定定义时的作用域，而不是调用时的作用域。
2. 箭头函数可以让this指向固定化，这种特性很有利于封装回调函数。

> this指向的固定化，并不是因为箭头函数内部有绑定this的机制，实际原因是箭头函数根本没有自己的this，导致内部的this就是外层代码块的this。正是因为它没有this，所以也就不能用作构造函数。


箭头函数实质上就是保存了其外层作用域的this指向，从而在函数体内进行调用。ES6与ES5的代码如下：
```
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}
// ES5
function foo() {
  var _this = this;
  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```

-----

### 绑定例外问题
1. 把null或undefined作为绑定对象传入call或者apply,此时实际绑定的是默认规则(全局对象).
> 这种方法主要是用apply展开数组并传入函数.但是如果这个函数确实使用了this,那么this会绑定到全局对象,造成恶劣后果.
2. “更安全”的做法是传入一个特殊的对象，把this绑定到这个对象不会对你的程序产生任何副作用
``` bash
function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
}
// 我们的DMZ空对象
var ø = Object.create( null );
// 把数组展开成参数
foo.apply( ø, [2, 3] ); // a:2, b:3
// 使用bind(..)进行柯里化
var bar = foo.bind( ø, 2 ); 
bar( 3 ); // a:2, b:3
```


>  参考：**《你不知道的JavaScript》**等