---
title: 箭头函数

tags:
 - ES6
categories:
 - 前端
 - JavaScript
 - ECMAScript 6
date: 2018-06-05 10:34:10
banner: https://static.oushiun.com/blog/banner/arrows.jpg
---

### 基本用法

胖箭头函数 **Fat arrow functions**，又称箭头函数，是一个来自 ECMAScript 2015（又称 ES6）的全新特性。有传闻说，箭头函数的语法 `=>`，是受到了 _CoffeeScript_ 的影响，并且与 _CoffeeScript_ 中的 `=>` 语法一样，共享 _this_ 上下文。

<!-- more -->

箭头函数的产生，主要由两个目的：更简洁的语法和与父作用域共享关键字 _this_。

```javascript
var f = v => v

// 等同于
var f = function(v) {
    return v
}
```

如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

```javascript
var f = () => 5
// 等同于
var f = function() {
    return 5
}

var sum = (num1, num2) => num1 + num2
// 等同于
var sum = function(num1, num2) {
    return num1 + num2
}
```

如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用 _return_ 语句返回。

```javascript
var sum = (num1, num2) => {
    return num1 + num2
}
```

由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号，否则会报错。

```javascript
// 报错
let getTempItem = id => { id: id, name: 'Temp' }

// 不报错
let getTempItem = id => ({ id: id, name: 'Temp' })
```

下面是一种特殊情况，虽然可以运行，但会得到错误的结果。

```javascript
let foo = () => {
    a: 1
}
foo() // undefined
```

上面代码中，原始意图是返回一个对象 `{ a: 1 }`，但是由于引擎认为大括号是代码块，所以执行了一行语句 `a: 1`。这时`a` 可以被解释为语句的标签，因此实际执行的语句是 `1`，然后函数就结束了，没有返回值。

如果箭头函数只有一行语句，且不需要返回值，可以采用下面的写法，就不用写大括号了。

```javascript
let fn = () => void doesNotReturn()
```

箭头函数可以与变量解构结合使用。

```javascript
const full = ({ first, last }) => first + ' ' + last

// 等同于
function full(person) {
    return person.first + ' ' + person.last
}
```

箭头函数使得表达更加简洁。

```javascript
const isEven = n => n % 2 == 0
const square = n => n * n
```

上面代码只用了两行，就定义了两个简单的工具函数。如果不用箭头函数，可能就要占用多行，而且还不如现在这样写醒目。

箭头函数的一个用处是简化回调函数。

```javascript
var list = [1, 2, 3]

// 正常函数写法
list.map(function(x) {
    return x * x
})

// 箭头函数写法
list.map(x => x * x)
```

```javascript
// 正常函数写法
var result = values.sort(function(a, b) {
    return a - b
})

// 箭头函数写法
var result = values.sort((a, b) => a - b)
```

下面是 _rest_ 参数与箭头函数结合的例子。

```javascript
const numbers = (...nums) => nums

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail]

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```

### this 指向

this 指向的固定化，并不是因为箭头函数内部有绑定 this 的机制，实际原因是箭头函数根本没有自己的 this，导致内部的 this 就是外层代码块的 this。正是因为它没有 this，所以也就不能用作构造函数。

```javascript
// ES6
function foo() {
    setTimeout(() => {
        console.log('id:', this.id)
    }, 100)
}

// ES5
function foo() {
    var _this = this

    setTimeout(function() {
        console.log('id:', _this.id)
    }, 100)
}
```

上面代码中，转换后的 es5 清楚地说明了，箭头函数里面根本没有自己的 this，而是引用外层的 this。

普通函数中的 this：

-   默认情况下（非严格模式），没有找到直接调用者，this 指向 window
-   严格模式（'use strict'），没有找到直接调用者，this 是 undefined
-   this 总是代表它的直接调用者，比如：obj.fun，那么 fun 中的 this 是 obj
-   使用 call，apply，bind 绑定的 this 指向的是绑定的对象

### 注意事项

箭头函数有几个使用注意点：

-   函数体内的 this 对象，就是定义时所在的对象，而不是使用时所在的对象；
-   不可以当作构造函数，也就是说，不可以使用 new 命令，否则会抛出一个错误；
-   不可以使用 arguments 对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替；
-   不可以使用 yield 命令，因此箭头函数不能用作 Generator 函数。

### 延伸阅读

[MDN Arrow Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
[Higher order functions in ES6:Easy as a => b => c;](https://developer.ibm.com/node/2016/01/11/higher-order-functions-in-es6easy-as-a-b-c/)
