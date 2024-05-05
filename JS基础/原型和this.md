# this
this是在运行时绑定的，完全取决于其调用位置，和其他OOP语言类似，this是在函数调用时候隐式传递的一个参数
## 隐式绑定
```javascript

function foo() {
	console.log( this.a );
}
var obj2 = {
	a: 42,
	foo: foo
};
var obj1 = {
	a: 2,
	obj2: obj2
};
obj1.obj2.foo(); // 42

//绑定丢失
var bar = obj2.foo;
bar() // undefined

```

## 显式绑定

* 使用call函数
```javascript
function foo() {
	console.log( this.a );
}
var obj = {
	a:2
};
foo.call( obj ); // 2

```
## 硬绑定
```javascript
function foo() {
	console.log( this.a );
}
var obj = {
	a:3
};
var bar = function() {
	foo.call(obj);
};
setTimeout(bar, 100); // 2
setTimeout(()=>{foo.call(obj);}, 100); // 2

//硬绑定后函数内部this无法更改
bar.call( window ); // 2
var obj2 = {
	fn: bar
}
obj2.bar() // 2
```
无论bar函数后续绑定在哪个对象中，内部foo函数的this都指向obj，bar函数就是一个硬绑定的函数

```javascript
// 简单的辅助绑定函数
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

//
Function.prototype.myBind = function(obj){
	let that = this
	return function(){
		that.apply(obj, arguments);
	}
}
//使用箭头函数
Function.prototype.myBind = function(obj){
	return (...args)=>{ //不定参数
		this.apply(obj, args)
	}
}

var obj = {
	a: 1
}
function bar(){
	console.log(this.a)
}	
bar.myBind(obj)()// 1
```
## new绑定
* 创建（或者说构造）一个全新的对象。
* 这个新对象会被执行[[ 原型]] 连接。
* 这个新对象会绑定到函数调用的this。
* 如果函数没有返回其他对象，那么new 表达式中的函数调用会自动返回这个新对象。
```javascript
function foo(a) {
	this.a = a;
}
var bar = new foo(2);
console.log( bar.a ); // 2

```

## 优先级
new绑定 > 显式绑定 > 隐式绑定 > 默认绑定

# 软绑定
硬绑定的问题是一旦bind之后无法修改函数内部的this
```javascript
Function.prototype.softBind = function(obj){
	let that = this
	return function(){
		if(!this || this == window){
			that.apply(obj, arguments);
		}else{
			that.apply(this, arguments);
		}
	}
}

var obj1 = {
	name: "obj1"
}	
function bar(){
	console.log(this.name)
}	
let bb = bar.softBind(obj1)
bb() //obj1

let obj2 = {
	bb: bb,
	name: "obj2"
}
//实现this的绑定转移
obj2.bb() // obj2

let obj3 = {
	name: "obj3"
}
bb.apply(obj3) // obj3

```

* 箭头函数没有this

## 原型
[[Prototype]]
JavaScript 中的对象有一个特殊的 [[Prototype]] 内置属性，其实就是对于其他对象的引
用。几乎所有的对象在创建时[[Prototype]] 属性都会被赋予一个非空的值。

```javascript
var anotherObject = {
	a:2
};
// 创建一个关联到anotherObject的对象
var myObject = Object.create( anotherObject );
myObject.a; // 2

//原型链，最后的都是Object.prototype
myObject
	|  [[Prototype]]
anotherObject
	|  [[Prototype]]
Object

var myObject2 = Object.create( anotherObject );
//修改原型的属性
Object.getPrototypeOf(myObject2).a = 3
myObject.a; // 3

```

所有的函数默认都会拥有一个名为prototype 的公有并且不可枚举的属性，它会指向另一个对象

```javascript
function Foo() {
// ...
}
var a = new Foo();
Object.getPrototypeOf(a) === Foo.prototype; // true
a.__proto__ === Object.getPrototypeOf(a) === Foo.prototype
a.constructor === Foo; // true
a instanceof Foo // true
//使用new之后
//构造一个对象a
//将a的[[Prototype]]绑定到Foo.prototype)
//a的constructor改成Foo

```

## 原型链
### js通过"原型链"模拟类的行为
```javascript

//属性&方法通过原型链查找

function Object1(){
	a: "a"
}	
Object1.prototype.fn = () => {
	console.log("fn")
}
let o1 = new Object1()
let o2 = new Object1()
console.log(o1.b) //a
console.log(o2.b) //a
console.log(o1.fn()) //fn
console.log(o2.fn()) //fn

//Object1.prototype = o1.__proto = o2.__proto 
//通过原型链向上查找可以找到b
Object1.prototype.b = 2
console.log(o1.b) //2
console.log(o2.b) //2

o1.c = 3
console.log(o1.c) //3
console.log(o2.c) //undefined

```
### 模拟继承
* 原型风格
```javascript
//模拟Bar继承Foo
Function.prototype.myBind = function(obj){
	return (...args)=>{ //不定参数
		this.apply(obj, args)
	}
}
function Foo(name) {
	this.name = name;
}
Foo.prototype.myName = function() {
	return this.name;
};
Foo.prototype.add = function() {
	console.log("Foo add")
};

function Bar(name,label) {
	//模拟下super
	this.super = Foo.myBind(this)
	this.super(name);
	this.label = label;
	this.add = ()=>{
		this.__proto__.add()//调用super类似
		console.log("Bar add")
	}
}
Bar.prototype = Object.create( Foo.prototype );//原型链
Bar.prototype.myLabel = function() {
	return this.label;
};
var a = new Bar( "a","obj a" );
console.log(a.myName()); // "a"
console.log(a.myLabel()); // "obj a"

a.add() //Foo add; Bar add

a
| 	__proto__
Bar.prototype
|	prototype
Foo.prototype
|	prototype
Object.prototype

//__proto__的近似实现
Object.defineProperty( Object.prototype,"__proto__", {
	get: function() {
		return Object.getPrototypeOf( this );
	},
	set: function(o) {
		// ES6中的setPrototypeOf(..)
		Object.setPrototypeOf( this, o );
		return o;
	}
});

```

## 通过委托风格模拟类继承
不通过构造函数创建对象
```javascript

Foo = {
	init: function(who) {
		this.me = who;
	},
	identify: function() {
		return "I am " + this.me;
	}
};
Bar = Object.create(Foo);
// Object.create === Bar.__proto__ = Foo
Bar.speak = function() {
	alert( "Hello," + this.identify() + "." );
};
//可以把Bar和Foo都理解成类对象？
var b1 = Object.create( Bar );
b1.init( "b1" );
console.log(b1) //{me: 'b1'}
var b2 = Object.create( Bar );
console.log(b2) //{}


```
