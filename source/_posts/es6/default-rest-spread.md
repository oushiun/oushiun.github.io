---
title: 默认参数、不定参数、扩展运算符

tags:
 - ES6
categories:
 - 前端
 - JavaScript
 - ECMAScript 6
date: 2018-06-20 11:02:29
banner: https://static.oushiun.com/blog/banner/default-rest-spread.jpg
---

### 默认参数(default)

这是一个简单的小增加，使得处理函数参数变得更容易。无论函数定义中声明的参数数量多少，JavaScript 中的函数都允许传递任意数量的参数。您可能知道当前 JavaScript 代码中**常见的模式**：

```javascript
function inc(number, increment) {
    // set default to 1 if increment not passed
    // (or passed as undefined)
    increment = increment || 1
    return number + increment
}

console.log(inc(2, 2)) // 4
console.log(inc(2)) // 3
```

当 _increment_ 为 `undefined` 或者不填写，通过逻辑**或操作符(`||`)**返回 1。

ES6 为我们提供了设置默认功能参数的方法。任何具有默认值的参数都被认为是可选的。

`inc` 函数的 ES6 代码如下所示：

```javascript
function inc(number, increment = 1) {
    return number + increment
}

console.log(inc(2, 2)) // 4
console.log(inc(2)) // 3
```

你还可以将默认值设置为出现在没有默认值的参数之前的参数：

```javascript
function sum(a, b = 2, c) {
    return a + b + c
}

console.log(sum(1, 5, 10)) // 16 -> b === 5
console.log(sum(1, undefined, 10)) // 13 -> b as default
```

你甚至可以执行一个函数来设置默认参数。所以它不局限于**原始值**。

```javascript
function getDefaultIncrement() {
    return 1
}
function inc(number, increment = getDefaultIncrement()) {
    return number + increment
}

console.log(inc(2, 2)) // 4
console.log(inc(2)) // 3
```

### 不定参数(rest)

重写 _sum_ 函数来处理传递给它的所有参数，如果使用 ES5，我们可能也要使用 `arguments` 对象。

```javascript
function sum() {
    var numbers = Array.prototype.slice.call(arguments),
        result = 0
    numbers.forEach(function(number) {
        result += number
    })
    return result
}

console.log(sum(1)) // 1
console.log(sum(1, 2, 3, 4, 5)) // 15
```

但是，该函数不能够处理任何参数。我们必须扫描函数的形体并找到参数对象。
ECMAScript 6 引入了不定参数(_rest_)来帮助我们解决这个问题和其他问题。

> arguments - contains all parameters including named parameters

REST 参数由三个点表示…在一个参数之前。命名参数成为包含其余参数的数组。

_sum_ 函数使用 ES6 语法重写：

```javascript
function sum(...numbers) {
    var result = 0
    numbers.forEach(function(number) {
        result += number
    })
    return result
}

console.log(sum(1)) // 1
console.log(sum(1, 2, 3, 4, 5)) // 15
```

> 注意：在函数声明中不能使用其他命名参数

```javascript
function sum(...numbers, last) {
    // causes a syntax error
    var result = 0
    numbers.forEach(function(number) {
        result += number
    })
    return result
}
```

### 扩展运算符(spread)

扩展运算符与不定参数密切相关，它允许分割数组单参数是作为单独的参数传递给函数。

通过 spread 定义 _sum_ 函数：

```javascript
function sum(a, b, c) {
    return a + b + c
}

var args = [1, 2, 3]
console.log(sum(...args)) // 6
```

等同于 ES5 写法：

```javascript
function sum(a, b, c) {
    return a + b + c
}

var args = [1, 2, 3]
console.log(sum.apply(undefined, args)) // 6
```

因此，使用 `apply` 函数，我们可以输入 `...args` 并单独传递所有数组参数。

我们也可以混合使用：

```javascript
function sum(a, b, c) {
    return a + b + c
}
var args = [1, 2]
console.log(sum(...args, 3)) // 6
```

结果是一样的。前两个参数来自 args 数组，最后一个参数是 3。
