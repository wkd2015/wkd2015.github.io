---
title: JavaScript中基于原型链的“继承”机制
date: 2018-04-15 23:30:03
tags: [JavaScript,ES6]
---


> 对于JavaScript中的“继承”机制的学习



<!-- more -->
### 传统面向对象语言中的类
#### 类/继承  
类是一种代码的组织结构，用于对真实世界中的问题建模。面向对象的语言强调的是数据与其行为相关联，提供封装了相关行为的结构，即数据结构。  
类的继承是指一个类继承了另一个类的基础定义，而通过类所创建的内容就是类的实例。
#### 类的多态性  
父类的通用行为可以被子类用更特殊的行为所重写。
#### 类的行为本质  
在传统的面向对象语言中，类的继承、实例化，实质上就是一种复制行为。  
子类相对于父类来说是一个独立并且完全不同的类。子类会包含父类行为的样本，也可以通过在子类中定义与父类中相同的方法名来改写某个继承的行为，这种改写不会影响父类的方法，两者互不影响。

-----

### JavaScript中的“类”

> 在继承和实例化的过程中，JavaScript的对象机制决定了并不会自动执行复制行为。简单来说，JavaScript中只有对象，并不存在可以被实例化的类。一个对象并不会被复制到其他对象，被复制的只是引用而已，它们通过这种关系实现关联。


#### 原型  
在JavaScript中，函数本身也是一个包含了方法和属性的对象。每个函数都有一个特殊的[[prototype]] 内置属性，其实就是对于其他对象的引用，它的初始值指向一个“空”对象。  
当我们访问构造器函数创建的函数的某个属性时，JavaScript引擎会遍历该对象的所有属性，并查找出该属性，如果找到了就会立即返回该值；如果找不到该属性，引擎就会去查询用于创建当前对象的构造器函数的原型，如果在原型中查找到了该属性，就立即使用该属性。因此我们**可以利用原型对构造器函数添加方法和属性**。  
由于在JavaScript中，几乎所有对象都是通过传引用的方式来传递的，因此我们所创建的每个新对象实体中并没有一份属于自己的原型副本。这也就意味着我们**可以随时修改prototype属性，并且由同一构造器创建的所有对象的prototype属性也都会同时改变，即使是在修改之前就已经创建的对象也一样**。
##### Tips
> - 原型中所有属性是被很多实例共享的，这种共享对于函数非常合适。但是对于包含引用类型的属性来说，问题就比较大了。例如在原型中定义一个包含数组的属性，那么在实例中对数组进行修改的话，其他示例中所引用的数组内容也会变化。
> - 如果遇上对象的自身属性与原型属性同名，那么对象自身属性的优先级高于原型属性。
>   - 可以利用自身属性重写原型属性。
>	- 可以通过hasOwnProperty()方法判断一个属性属于自身属性还是原型属性。


#### 原型链
由原型的定义知道，JavaScript中每一个函数都有一个指向某一对象的prototype属性，该函数被new操作符调用时会创建并返回一个对象，并且该对象中会有一个指向其原型对象的链接，我们可以**在新建的对象中调用相关原型对象的方法和属性。而原型对象本身也有对象固有的普遍特征，因此本身也有指向其原型的链接，由此就形成了一条链**。  

#### “继承”
> 基础思想: 利用原型链的调用特性实现一个引用类型继承另一个引用类型的属性和方法。



在传统的面向类的语言中，类可以被复制多次，每次实例化的过程都是一次复制。而JavaScript是一种完全依靠对象的语言，没有类似的复制机制。你不能创建一个类的多个实例，只能创建多个对象，他们的[[prototype]]关联的是同一个对象。因为在默认情况下，并不会进行复制，所以这些对象之间并不会完全失去联系，他们是互相关联的。    
在JavaScript中，称为“继承”其实是不准确的，因为传统面向类的语言中继承意味着复制操作，而**JavaScript默认情况下并不会复制对象属性，而是在两个对象之间创建一个关联，这样一个对象可以委托访问另一个对象的属性和函数**。委托可以更加准确的描述JavaScript中对象的关联机制。

##### 构造函数继承
> 借助构造函数以及prototype实现继承的方式



```
function Parent () {
    this.name = "parent";
}
Parent.prototype.say = function () {
	console.log(this.name);   
} 
function Child () {
	Parent.call(this);
	this.type = "child";
}
```
* 问题
    * 这种方法只能继承父类构造函数中声明的实例属性，并没有继承父类原型中定义的属性和方法
    * 原因在于call方法的本质是改变了当前函数的执行上下文，只能保证原本父类中this指向的属性在call操作后被复制到当前this上，而未对原型上的方法做操作。  
![](https://raw.githubusercontent.com/wkd2015/Pic/master/blog/prototype_1.jpg)
##### 原型链继承
> 修改原型链指向以实现继承



```
function Parent () {
    this.name = "parent";
	this.code = [1,2,3];
}
Parent.prototype.say = function () {
	console.log(this.name);   
} 
function Child () {
	this.type = "child";
}
Child.prototype = new Parent();
s1 = new Child();
s2 = new Child();
```
* 问题
    * **所有实例共享原型对象中的所有属性和方法**，共享是这种方法的最大优点，也是它的最大缺点。大多数情况下我们常常需要某些属性是子类特有的，而一些读取属性方法才需要共享。例如上例中的code数组，实际上两个实例引用的是同一个数组。  
![](https://raw.githubusercontent.com/wkd2015/Pic/master/blog/prototype_2.jpg)

##### 组合继承
> - 将原型链和借用构造函数的技术结合到一起的集成模式。
> - **原理**：使用原型链实现对共有的原型属性和方法的继承，通过call方法借用构造函数来实现对实例独有属性的继承。这样，即通过在原型上定义方法实现了函数复用，又能保证每个实例都有它的属性。



```
function Parent () {
    this.name = "parent";
	this.code = [1,2,3];
}
Parent.prototype.say = function () {
	console.log(this.name);   
} 
function Child () {
	Parent.call(this);
	this.type = "child";
}
Child.prototype = new Parent();
var s1 = new Child();
var s2 = new Child();
s1.code === s2.code;  //false
```
* 问题
    * 子类实例化的时候，父类构造函数被执行了两次
        * 利用new操作符调用父类构造函数时执行一次
        * 在子类中借用call调用父类构造函数时执行一次

##### 寄生组合式继承
> 本质上来说，组合式继承中之所以在子类中借用父类构造函数，是为了实现对父类中属性的复制操作，而基于这一思想，我们可以通过变通实现与new操作符调用父类构造函数一样的效果，即**浅拷贝**。



```
function Parent () {
    this.name = "parent";
	this.code = [1,2,3];
}
Parent.prototype.say = function () {
	console.log(this.name);   
} 
function Child () {
	Parent.call(this);
	this.type = "child";
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
var s1 = new Child();
var s2 = new Child();
```
##### ES6中Class的继承
> ES6中引入的Class实质上是JavaScript现有的基于原型的继承的语法糖。类语法不会为JavaScript引入新的面向对象的继承模型。



- Class的继承方法是extend关键字  
    - extends关键字在类声明或类表达式中用于创建一个类作为另一个类的一个子类。
    - **子类的构造函数必须执行一次super函数**。
```
class Animal { 
    constructor(name) {
        this.name = name;
    }
    speak() {
        console.log(this.name + ' makes a noise.');
    }
}
class Dog extends Animal {
    constructor() {
        super();
    }
    speak() {
        console.log(this.name + ' barks.');
    }
}
var d = new Dog('Mitzie');
d.speak();
```