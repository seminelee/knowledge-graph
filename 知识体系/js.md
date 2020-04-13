# 4. js

## 4.1 JavaScript执行上下文、作用域和闭包

[JavaScript执行上下文、作用域和闭包](https://juejin.im/post/5d4fd2ed6fb9a06b1777a6c3)



### 4.1.1 JS中的栈、队列和堆

- 执行上下文栈：执行的时候首先是全局上下文入栈，然后各种函数上下文入栈，当函数全部执行完之后，最后全局执行上下文出栈

```js
let fn, bar; // 1、进入全局上下文环境
bar = function(x) {
	let b = 5;
	fn(x + b); // 3、进入fn函数上下文环境
};
fn = function(y) {
	let c = 5;
	console.log(y + c); //4、fn出栈，bar出栈
};
bar(10); // 2、进入bar函数上下文环境
```

- 堆：存储对象这种复杂的数据类型（引用数据类型）
- 异步执行队列：存储我们在代码中书写的异步操作的callback，用于event loop的执行。

> js数据类型：
>
> - 值类型(基本类型)：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。
> - 引用数据类型：对象(Object)（数组、函数都属于对象类型）。



### 4.1.2  作用域和this

__作用域__

JavaScript中有全局作用域，函数作用域，但没有块级作用域。

```js
var a = 10;//全局作用域中定义变量
(function(){
	var b = 20;//函数作用域中定义变量
})();
console.log(a);//可以访问全局变量
console.log(b);//error, b is not defined
```

JavaScript是基于 __词法作用域__ 来创建作用域的，也就是说一个函数的作用域在函数声明的时候（即在函数执行上下文创建的时候）就已经定义好的，不是函数执行的时候。（区别于动态作用域）

```js
var scope = 'global scope';

function f() {
	console.log(scope) // 基于词法作用域，声明时已经定义好作用域，所以是global scope
}

function checkscope() {
	var scope = 'local scope';
	f();
}
checkscope();
```

作用域链：当一个块或者函数嵌套在另一个块或函数中时，就发生了作用域的嵌套。在当前函数中如果JS引擎无法找到某个变量，就会往上级嵌套的作用域中去寻找，直到找到该变量或抵达全局作用域，这样的链式关系成为作用域链（Scope Chain）。



__`this`__
 `this`就是上下文对象，即被调用函数所处的环境，也就是说，`this` 在函数内部指向了调用函数的对象。 

- 在全局环境下，this 始终指向全局对象（window）
- 普通函数内部的this：非严格模式下，this 默认指向全局对象window；而严格模式下， this为undefined
- 对象内部方法的this指向调用这些方法的对象
- 构造函数中的this与被创建的新对象绑定



### 4.1.3 执行上下文

在函数执行之前，会为当前函数创建一个执行上下文，并且在这时会创建一个**变量对象**。 其中上下文对象就是一个包括三个属性的对象,三个属性分别是，**变量对象**,**作用域链**,**this对象**

例子说明：

``` js
function demo(num) {
	var name = 'changan';
	var getData = function getData(){}
	function c(){}
}
demo(100)
```

创建阶段

``` js
executionContext = {
	scopeChain: [VO, globalContext.VO], // 在创建阶段已经定义好作用域，VO即variable object
	variableObject: {
		arguments: { // 创建参数对象
			0: 100,
			length: 1
		},
		num: 100,// 创建形参名称，赋值或者创建引用拷贝
		c: function c(){}, // 有内部函数声明的话，创建引用指向函数体
		name: undefined, // 有内部声明变量a，初始化为undefined
		getData: undefined // 有内部声明变量getData，初始化为undefined
	},
	this: {...}
}
```

代码执行阶段

``` js
executionContext: {
	scopeChain: [VO, globalContext.VO],
	variableObject: {
		arguments: {
			0: 100,
			length: 1
		},
		num: 100,
		c: function c() {},
		name: 'changan', // 分配变量赋值
		getData: function getData() {} // 分配函数引用，赋值
	},
	this: {...}
}
```

执行时，在当前函数中如果JS引擎无法找到某个变量，就会往上级嵌套的作用域中去寻找，直到找到该变量或抵达全局作用域，作用域链。



### 4.1.4 词法环境和闭包

[JavaScript学习笔记之闭包](https://seminelee.github.io/2016/08/06/bibao/)



__词法环境__

词法环境由两部分组成：

- Environment Records（环境记录）：这个就是变量登记的地方了 （变量对象）
- outer:outer 是个指向，包含（包围）本词法环境的外部词法环境 （作用域链）

上面说到执行上下文（执行环境）包括变量对象（变量环境组件）、作用域链（词法环境组件）、this对象。

| 组件         | 作用目的                                                     |
| ------------ | ------------------------------------------------------------ |
| 词法环境组件 | 指定一个[词法环境](https://www.w3.org/html/ig/zh/wiki/ES5/可执行代码与执行环境#lexical-environment)对象，用于解析该执行环境内的代码创建的[标识符](https://www.w3.org/html/ig/zh/wiki/ES5/lexical#x7.6)引用。 |
| 变量环境组件 | 指定一个[词法环境](https://www.w3.org/html/ig/zh/wiki/ES5/可执行代码与执行环境#lexical-environment)对象，其环境数据用于保存由该执行环境内的代码通过 *VariableStatement* 和 *FunctionDeclaration* 创建的绑定。 |
| this 绑定    | 指定该执行环境内的 ECMA 脚本代码中 **this** 关键字所关联的值。 |



__闭包__

闭包是函数和声明该函数的词法环境的组合。**这个环境包含了这个闭包创建时所能访问的所有局部变量**。

在JavaScript中，函数也是对象，并且函数也可以作为返回值。闭包，不同于一般的函数，它允许一个函数在立即词法作用域外调用时，仍可访问非本地变量。

```js
function outer(){
	var localVal = 30;
	return function(){
		return localVal;
	}
}
var func = outer();
func();//30
```

用处：用闭包模拟私有方法或私有变量

问题：局部变量没有被释放掉造成空间浪费，性能消耗



### 4.1.5__`const` `let` 和箭头函数__

`let`、`const`语句不出现变量提升

- `let`：所声明的变量，只在`let`命令所在的代码块内有效（块级作用域）

``` js
for (let i = 0; i < 10; i++) {
// ...
}
console.log(i);
// ReferenceError: i is not defined
```

- `const` 声明一个只读的常量。一旦声明，常量的值就不能改变。`const`的作用域与`let`命令相同：只在声明所在的块级作用域内有效。

- 箭头函数：更简短的函数并且不绑定`this`。箭头函数不会创建自己的`this`,它只会从自己的作用域链的上一层继承`this`（因此不能作为构造函数）

## 4.2 事件机制

[事件介绍](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Building_blocks/Events)

JavaScript 事件触发有三个阶段。

- `CAPTURING_PHASE`，即捕获阶段
- `AT_TARGET`，即目标阶段
- `BUBBLING_PHASE`，即冒泡阶段

 在捕获阶段：

- 浏览器检查元素的最外层祖先`<html>`，是否在捕获阶段中注册了一个`onclick`事件处理程序，如果是，则运行它。
- 然后，它移动到`<html>`中单击元素的下一个祖先元素，并执行相同的操作，然后是单击元素再下一个祖先元素，依此类推，直到到达实际点击的元素。

在冒泡阶段，恰恰相反:

- 浏览器检查实际点击的元素是否在冒泡阶段中注册了一个`onclick`事件处理程序，如果是，则运行它
- 然后它移动到下一个直接的祖先元素，并做同样的事情，然后是下一个，等等，直到它到达`html`元素。

### 4.2.1 事件监听

刚刚说的注册事件实际上就是绑定事件/事件监听

``` js
target.addEventListener(type, listener [, useCapture]);
```

通常我们使用 `addEventListener` 注册事件，该函数有一个 `useCapture` 参数，该参数接收一个布尔值，默认值为 `false` ，代表注册事件为冒泡事件。若想注册事件为捕获事件，则将 `useCapture` 设置为 `true` 。

### 4.2.2 禁止冒泡

``` js
video.onclick = function(e) {
  e.stopPropagation(); // 只会让当前事件处理程序运行，但事件不会在冒泡链上进一步扩大，因此将不会有更多事件处理器被运行(不会向上冒泡)。
  video.play();
}
```

### 4.2.3 事件委托

冒泡还允许我们利用事件委托——这个概念依赖于这样一个事实,如果你想要在大量子元素中单击任何一个都可以运行一段代码，您可以将事件监听器设置在其父节点上，并让子节点上发生的事件冒泡到父节点上，而不是每个子节点单独设置事件监听器。

## 4.3 `bind`和 `apply`和`call`

### 4.3.1 定义

- `function.bind(thisArg[, arg1[, arg2[, ...]]])`

`bind()`方法创建一个新的函数，在`bind()`被调用时，这个新函数的`this`被bind的第一个参数指定，其余的参数将作为新函数的参数供调用时使用。【返回值是函数，参数以参数列表传递】

- `func.apply(thisArg, [argsArray])`

`apply()`方法调用一个具有给定`this`值的函数，以及作为一个数组（或[类似数组对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Indexed_collections#Working_with_array-like_objects)）提供的参数。【返回值是执行结果，参数以数组形式传递】

- `fun.call(thisArg, arg1, arg2, ...)`

与 [`apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法类似，只有一个区别，就是 `call()` 方法接受的是**一个参数列表**，而 `apply()` 方法接受的是**一个包含多个参数的数组**。【返回值是执行结果，参数以参数列表传递】

### 4.3.2 实现

bind

https://github.com/seminelee/es6/blob/master/bind.js

``` js
Function.prototype.myBind = function(thisArg, ...argArray) {
  var BoundTargetFunction = this
  var boundThis = thisArg
  function bindFn() {
    return BoundTargetFunction.call(boundThis, ...argArray)
  }
  return bindFn
}
```

apply

``` js
Function.prototype.myApply = function(thisArg, argArray) {
  if (thisArg) {
    thisArg.fn = this // 把要调用的函数挂载在thisArg对象上
    const result = thisArg.fn(...argArray)
    delete thisArg.fn
    return result
  } else {
    return this(...argArray)
  }
}
```

call

``` js
Function.prototype.myCall = function(thisArg, ...argArray) {
  if (thisArg) {
    thisArg.fn = this // 把要调用的函数挂载在thisArg对象上
    const result = thisArg.fn(...argArray)
    // 不用es6的写法可以这样写
    // arguments.shift()
    // eval('context.fn(' + arguments +')')
    delete thisArg.fn
    return result
  } else {
    return this(...argArray)
  }
}
```



## 4.4 原型链

### 4.4.1 原型链

[JavaScript学习笔记之对象及继承](https://seminelee.github.io/2016/08/07/ob/)

创建对象的模式：

组合使用 __构造函数模式__ 与__原型模式__ 是更好的选择。即在构造函数中定义实例属性，而所有实例共享的属性在原型中定义。

> 设计模式：
>
> 发布订阅模式、工厂模式（创建一个对象并返回）、代理模式（`Proxy`）、装饰者模式（ `@decoratorClass`）
>
> 代理模式：在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截
>
> 装饰者模式： 可以动态的给某个对象添加额外的职责，而不会影响从这个类中派生的其它对象`

JS主要靠原型链来实现继承

每个实例对象（ object ）都有一个私有属性（称之为 __proto__ ）指向它的构造函数的原型对象（**prototype** ）。该原型对象也有一个自己的原型对象( __proto__ ) ，层层向上直到一个对象的原型对象为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**中的最后一个环节。

```js
function SuperType(){
this.property = true;
}
SuperType.prototype.getSuperValue = function(){
return this.property;
}
function SubType(){
this.subProperty = false;
}
//继承了SuperType
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function(){
return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue()); //true
```

即`instance.__proto__`（instance的原型）指向`SubType.prototype`，`SubType.prototype.__proto__`指向`SuperType.prototype`，而`SuperType.prototype.__proto__`指向`Object.prototype`，最后`Object.prototype.__proto__`指向null。

``` js
instance.__proto__ === SubType.prototype // var instance = new SubType()
SubType.prototype.__proto__ === SuperType.prototype // SubType.prototype = new SuperType()
SuperType.prototype.__proto__ === Object.prototype
Object.prototype.__proto__ === null
```

``` js
SubType.__proto__ === Function.prototype
Function.prototype.__proto__ === Object.prototype
```

### 4.4.2 `new`__和`instanceof `__

__`new`关键字__

实现步骤：

1. 创建一个新对象，同时继承对象类的原型，即Person.prototype；
2. 执行对象类的构造函数，同时该实例的属性和方法被this所引用，即this指向新构造的实例；
3. 如果构造函数return了一个新的“对象”，那么这个对象就会取代整个new出来的结果。如果构造函数没有return对象，那么就会返回步骤1所创建的对象，即隐式返回this。

``` js
// let p = new Person()
let p = (function () {
    let obj = {};
    obj.__proto__ = Person.prototype;
    // 其他赋值语句...
    return obj;
})();
```

__`instanceof `__

  假设现在有 x instanceof y 一条语句，则其内部实际做了如下判断： 

``` js
while(x.__proto__! == null) {
    if(x.__proto__ === y.prototype) {
        return true;
    }
    x.__proto__ = x.__proto__.proto__;
}
if(x.__proto__ == null) {return false;}
```

即判断`x.__proto__ === y.prototype`



### 4.4.3 class

完全可以看作构造函数的另一种写法

事实上，类的所有方法都定义在类的`prototype`属性上面，都是原型方法（公有方法）。私有方法通过前面加上下划线如`_bar`模拟实现。

``` js
class Point {
constructor(x, y) {
this.x = x; // 实例属性
this.y = y;
}

toString() {
return '(' + this.x + ', ' + this.y + ')';
}
}
Point.prototype.constructor === Point // true
Object.keys(Point.prototype)
// [] 类的内部所有定义的方法，都是不可枚举的（non-enumerable）
```

相当于

``` js
function Point(x, y) {
this.x = x;
this.y = y;
}

Point.prototype.toString = function () {
return '(' + this.x + ', ' + this.y + ')';
};

Point.prototype.constructor === Point // true
Object.keys(Point.prototype)
// ["toString"]
```

class的继承

``` js
class Point {
}

class ColorPoint extends Point {
}
```



## 4.5 `Promise`和`async/await`

https://seminelee.github.io/2017/12/16/promise-2/

``` js
new Promise( function(resolve, reject) {...} /* executor */  );
```

executor是带有 `resolve` 和 `reject` 两个参数的函数 。Promise构造函数执行时立即调用`executor` 函数， `resolve` 和 `reject` 两个函数作为参数传递给`executor`（executor 函数在Promise构造函数返回所建promise实例对象前被调用）。`resolve` 和 `reject` 函数被调用时，分别将promise的状态改为*fulfilled（*完成）或rejected（失败）。executor 内部通常会执行一些异步操作，一旦异步操作执行完毕(可能成功/失败)，要么调用resolve函数来将promise状态改成*fulfilled*，要么调用`reject` 函数将promise的状态改为rejected。如果在executor函数中抛出一个错误，那么该promise 状态为rejected。

当最外层的 promise 状态改变时，遍历它的 queue 数组调用对应的回调，设置子 promise 的 status 和 value 并遍历它的 queue 数组调用对应的回调，然后设置孙 promise 的 status 和 value 并遍历它的 queue 数组调用对应的回调......依次类推

### 4.5.1 `Promise`

回调函数变成了链式写法，使程序流程变得很清楚

Promise的实现

https://github.com/seminelee/es6/blob/master/promise.js

### 4.5.2 `async/await`

async 函数是什么？一句话，它就是 Generator 函数的语法糖

`async`函数的返回值是 Promise 对象

``` js
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

它能使得我们在编写异步代码时变得像同步一样的方式来编写



## 4.6 `export`和`module.exports`

### 4.6.1 CommonJS模块规范 `module.exports`

``` js
var x = 5;
var addX = function (value) {
  return value + x;
};
module.exports.x = x;
module.exports.addX = addX;
```

`require`加载模块

``` js
var example = require('./example.js');

console.log(example.x); // 5
console.log(example.addX(1)); // 6
```

`exports`即`module.exports`  。我们可以直接在 exports 对象上添加方法，表示对外输出的接口，如同在module.exports上添加一样。这等同在每个模块头部，有一行这样的命令：

``` js
var exports = module.exports
var x = 5;
var addX = function (value) {
  return value + x;
};
exports.x = x;
exports.addX = addX;
```

不能直接将exports变量指向一个值，因为这样等于切断了exports与module.exports的联系。

### 4.6.2 ES6模块规范

不同于CommonJS，ES6使用 export 和 import 来导出、导入模块

``` js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year}
```

相当于

``` js
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

引入

``` js
import { firstName, lastName, year } from './profile.js'
```

默认输出__`export default`__和正常输出`export`

```javascript
// 第一组
export default function crc32() { // 输出
  // ...
}
import crc32 from 'crc32'; // 输入

// 第二组
export function crc32() { // 输出
  // ...
};
import {crc32} from 'crc32'; // 输入
```





## 4.7 nodejs与浏览器中的事件循环

https://seminelee.github.io/2019/01/26/event-loop/

event loop即事件循环，是指浏览器或Node的一种解决javaScript单线程运行时不会阻塞的一种机制，也就是我们经常使用异步的原理。

同步任务与异步任务 

### 4.7.1 浏览器中的事件循环

宏任务和微任务

- MacroTask（宏任务）包括script全部代码、setTimeout、setInterval、I/O、UI Rendering等；
- MicroTask（微任务）包括Process.nextTick（Node独有）、Promise、Object.observe(废弃)等。

执行步骤：

1. 整体script作为第一个宏任务进入主线程。
2. 同步任务被放到执行栈，异步任务会进入Event Table并注册函数，其回调函数按类别被放到宏任务队列和微任务队列中。
3. 执行完所有同步任务后，开始读取任务队列中的结果。检查微任务队列，如果有任务则按顺序执行。
4. 执行完所有微任务后，开始下一个宏任务。如此循环，直到两个队列（宏任务队列和微任务队列）的任务都执行完。

> **Event Table** 就是个注册站：主线程让Event Table注册一个函数，该函数会在5秒之后被调用。当指定的事情发生时，Event Table会将这个函数（回调函数）移到任务队列。

### 4.7.2 NodeJS中的事件循环

Node.js 最大的特点就是使用 **异步式 I/O** 与 **事件驱动** 的架构设计

Node.js的运行机制如下:

- V8 引擎解析 JavaScript 脚本。
- 解析后的代码，调用 Node API。
- libuv 库负责 Node API 的执行。它将不同的任务分配给不同的线程，形成一个 Event Loop（事件循环），以异步的方式将任务的执行结果返回给 V8 引擎。
- V8 引擎再将结果返回给用户。

6个阶段

- timers: 执行setTimeout和setInterval中到期的callback。
- pending callback: 上一轮循环中少数的callback会放在这一阶段执行。
- idle, prepare: 仅在内部使用。
- poll: 最重要的阶段，执行pending callback，在适当的情况下会阻塞在这个阶段。
- check: 执行setImmediate的callback。
- close callbacks: 执行close事件的callback，例如socket.on(‘close’[,fn])或者http.server.on(‘close, fn)。

event loop的每一次循环都需要依次经过上述的阶段。每个阶段都有自己的FIFO的callback队列（在timer阶段其实使用一个最小堆而不是队列来保存所有元素，比如timeout的callback是按照超时时间的顺序来调用的，并不是先进先出的队列逻辑），每当进入某个阶段，都会从所属的队列中取出callback来执行。当队列为空或者被执行callback的数量达到系统的最大数量时，进入下一阶段。这六个阶段都执行完毕称为一轮循环。

### 4.7.3 Nodejs与浏览器的Event Loop差异

- Node 端，microtask 在事件循环的各个阶段之间执行
- 浏览器端，microtask 在事件循环的 macrotask 执行完之后执行

## 4.8 typescript的优点

typescript是javascript的超集，本质上是向JavaScript增加静态类型系统。

> 增加**静态**这个定语，是为了和运行时的类型检查机制加以区分，强调静态类型系统是在**编译时**进行类型分析。

团队开发时很有利。编译时就能知道些可以避免的错误。

## 4.9 跨域及解决方案

https://seminelee.github.io/2016/09/20/ajax/

如果协议、域名和端口其中有一个不同，就被当作是跨域。

解决方案：

- jsonp：由于`<img>`、`<script>`都能不受限制地从其他域加载资源，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制。只能用于get请求

- CORS(Cross Origin Resource Share)

  ``` php
  $origin=isset($_SERVER['HTTP_ORIGIN'])?$_SERVER['HTTP_ORIGIN']:'';
  /************** 获取客户端的Origin域名 **************/
  header('Access-Control-Allow-Origin:'.$origin);
  ```

- websocket 该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。这种协议专门为快速传输小数据设计，适合用于移动端。

- 开发环境时配置proxy [http-proxy-middleware](https://link.jianshu.com/?t=https://github.com/chimurai/http-proxy-middleware#options)`实现

- 通过后台服务器转发请求

## 4.10 设计模式

- 工厂模式

常见的实例化对象模式，工厂模式就相当于创建实例对象的new，提供一个创建对象的接口。应用场景：JQuery中的$、Vue.component异步组件、React.createElement等

``` js
 // 某个需要创建的具体对象
    class Product {
        constructor (name) {
            this.name = name;
        }
        init () {}
    }
    // 工厂对象
    class Creator {
        create (name) {
            return new Product(name);
        }
    }
    const creator = new Creator();
    const p = creator.create(); // 通过工厂对象创建出来的具体对象
```

- 单例模式

保证一个类仅有一个实例，并提供一个访问它的全局访问点，一般登录、购物车等都是一个单例。应用场景：JQuery中的$、Vuex中的Store、Redux中的Store等

``` js
 // 单例对象
    class SingleObject {
        login () {}
    }
    // 访问方法
    SingleObject.getInstance = (function () {
        let instance;
        return function () {
            if (!instance) {
                instance = new SingleObject();
            }
            return instance;
        }
    })()
    const obj1 = SingleObject.getInstance();
    const obj2 = SingleObject.getInstance();
    console.log(obj1 === obj2); // true
```

- 观察者模式

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知

``` js
class Subject {
  constructor () {
    this.state = 0;
    this.observers = [];
  }
  getState () {
    return this.state;
  }
  setState (state) {
    this.state = state;
    this.notify();
  }
  notify () {
    this.observers.forEach(observer => {
      observer.update();
    })
  }
  attach (observer) {
    this.observers.push(observer);
  }
}

class Observer {
  constructor (name, subject) {
    this.name = name;
    this.subject = subject;
    this.subject.attach(this);
  }
  update () {
    console.log(`${this.name} update, state: ${this.subject.getState()}`);
  }
}

let sub = new Subject();
let observer1 = new Observer('o1', sub);
let observer2 = new Observer('o2', sub);

sub.setState(1);
```

- 发布/订阅模式

**观察者模式与发布/订阅模式区别: 本质上的区别是调度的地方不同**

虽然两种模式都存在订阅者和发布者（具体观察者可认为是订阅者、具体目标可认为是发布者），但是观察者模式是由具体目标调度的，而发布/订阅模式是统一由调度中心调的，所以观察者模式的订阅者与发布者之间是存在依赖的，而发布/订阅模式则不会。
