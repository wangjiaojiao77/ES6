# 1、let
## （1）基本用法
ES6 新增了let命令，用来声明变量。它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。
```
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```
## （2）不存在变量提升
let命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。
```
// var 的情况1
var foo;
console.log(foo); // 输出2
foo = 2;

// var 的情况2
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```

## （3）暂时性死区
在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。
```
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

## （4）不允许重复声明
let不允许在相同作用域内，重复声明同一个变量。
```
// 报错
function () {
  let a = 10;
  var a = 1;
}

// 报错
function () {
  let a = 10;
  let a = 1;
}
```

# 2、ES6的块级作用域
let实际上为 JavaScript 新增了块级作用域。

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。
```
// IIFE 写法
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法
{
  let tmp = ...;
  ...
}
```

# 3、do表达式
本质上，块级作用域是一个语句，将多个操作封装在一起，没有返回值。
```
{
  let t = f();
  t = t * t + 1;
}
```
上面代码中，块级作用域将两个语句封装在一起。但是，在块级作用域以外，没有办法得到t的值，因为块级作用域不返回值，除非t是全局变量。

现在有一个提案，使得块级作用域可以变为表达式，也就是说可以返回值，办法就是在块级作用域之前加上do，使它变为do表达式。
```
let x = do {
  let t = f();
  t * t + 1;
};
```
上面代码中，变量x会得到整个块级作用域的返回值。

# 4、const
### （1）const声明一个只读的常量。一旦声明，常量的值就不能改变。
```
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

### （2）const一旦声明变量，就必须立即初始化，不能留到以后赋值。
```
const foo;
// SyntaxError: Missing initializer in const declaration
```

### （3）const的作用域与let命令相同：只在声明所在的块级作用域内有效。
```
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

### （4）const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。
```
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}
```
### （5）const声明的常量，也与let一样不可重复声明。
```
var message = "Hello!";
let age = 25;

// 以下两行都会报错
const message = "Goodbye!";
const age = 30;
```

# 5、ES6声明变量的六种方法
ES5 只有两种声明变量的方法：var命令和function命令。ES6除了添加let和const命令，后面章节还会提到，另外两种声明变量的方法：import命令和class命令。所以，ES6 一共有6种声明变量的方法。

# 6、顶层对象的属性

在ES6中，为了保持兼容性，一方面规定，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩。
```
var a = 1;
// 如果在Node的REPL环境，可以写成global.a
// 或者采用通用方法，写成this.a
window.a // 1

let b = 1;
window.b // undefined
```

# 7、global对象
ES5 的顶层对象，本身也是一个问题，因为它在各种实现里面是不统一的。

- 浏览器里面，顶层对象是window，但 Node 和 Web Worker 没有window。
- 浏览器和 Web Worker 里面，self也指向顶层对象，但是 Node 没有self。
- Node 里面，顶层对象是global，但其他环境都不支持。

同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用this变量，但是有局限性。

- 全局环境中，this会返回顶层对象。但是，Node 模块和 ES6 模块中，this返回的是当前模块。
- 函数里面的this，如果函数不是作为对象的方法运行，而是单纯作为函数运行，this会指向顶层对象。但是，严格模式下，这时this会返回undefined。
- 不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了CSP（Content Security Policy，内容安全政策），那么eval、new Function这些方法都可能无法使用。

综上所述，很难找到一种方法，可以在所有情况下，都取到顶层对象。下面是两种勉强可以使用的方法。
```
// 方法一
(typeof window !== 'undefined'
   ? window
   : (typeof process === 'object' &&
      typeof require === 'function' &&
      typeof global === 'object')
     ? global
     : this);

// 方法二
var getGlobal = function () {
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```
现在有一个提案，在语言标准的层面，引入global作为顶层对象。也就是说，在所有环境下，global都是存在的，都可以从它拿到顶层对象。

垫片库system.global模拟了这个提案，可以在所有环境拿到global。

```
// CommonJS 的写法
require('system.global/shim')();

// ES6 模块的写法
import shim from 'system.global/shim'; shim();
```
上面代码可以保证各种环境里面，global对象都是存在的。

```
// CommonJS 的写法
var global = require('system.global')();

// ES6 模块的写法
import getGlobal from 'system.global';
const global = getGlobal();
```

上面代码将顶层对象放入变量global。
















