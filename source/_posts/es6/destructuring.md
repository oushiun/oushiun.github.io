---
title: 解构

toc: true
tags:
 - ES6
categories:
 - 前端
 - JavaScript
 - ECMAScript 6
date: 2018-06-19 16:07:48
thumbnail: https://static.oushiun.com/blog/banner/destructuring.jpg
---

### 什么是解构赋值？

**解构赋值允许你使用类似数组或对象字面量的语法将数组和对象的属性赋给各种变量**。这种赋值语法极度简洁，同时还比传统的属性访问方法更为清晰。

通常来说，你很可能这样访问数组中的前三个元素：

``` javascript
var first = someArray[0]
var second = someArray[1]
var third = someArray[2]
```

如果使用解构赋值的特性，将会使等效的代码变得更加简洁并且可读性更高：

``` javascript
var [first, second, third] = someArray
```

<!-- more -->

_SpiderMonkey_（Firefox 的 JavaScript 引擎）已经支持解构的大部分功能，但是仍不健全。你可以通过 [bug 694100](https://bugzilla.mozilla.org/show_bug.cgi?id=694100) 跟踪解构和其它 ES6 特性在 SpiderMonkey 中的支持情况。

### 数组与迭代器的解构

以上是数组解构赋值的一个简单示例，其语法的一般形式为：

``` javascript
[ variable1, variable2, ..., variableN ] = array
```

这将为 _variable1_ 到 _variableN_ 的变量赋予数组中相应元素项的值。如果你想在*赋值的同时声明变量*，可在赋值语句前加入 `var` 、`let` 或 `const` 关键字，例如：

``` javascript
var [ variable1, variable2, ..., variableN ] = array
let [ variable1, variable2, ..., variableN ] = array
const [ variable1, variable2, ..., variableN ] = array
```

事实上，用变量来描述并不恰当，因为你可以对任意深度的嵌套数组进行解构：

``` javascript
var [foo, [[bar], baz]] = [1, [[2], 3]]

console.log(foo) // 1
console.log(bar) // 2
console.log(baz) // 3
```

此外，你可以在对应位留空来跳过被解构数组中的某些元素：

``` javascript
var [, , third] = ['foo', 'bar', 'baz']

console.log(third) // "baz"
```

而且你还可以通过“_不定参数_”模式捕获数组中的所有尾随元素：

``` javascript
var [head, ...tail] = [1, 2, 3, 4]

console.log(tail) // [2, 3, 4]
```

当访问空数组或越界访问数组时，对其解构与对其索引的行为一致，最终得到的结果都是：`undefined`。

``` javascript
console.log([][0]) // undefined

var [missing] = []
console.log(missing) // undefined
```

### 解构对象

在对象上进行解构可以将变量绑定到对象的不同属性。你指定要绑定的属性，然后指定要绑定其值的变量。

``` javascript
var robotA = { name: 'Bender' }
var robotB = { name: 'Flexo' }

var { name: nameA } = robotA
var { name: nameB } = robotB

console.log(nameA) // "Bender"
console.log(nameB) // "Flexo"
```

**当属性和变量名称相同时**，有一个有用的语法快捷方式：

``` javascript
var { foo, bar } = { foo: 'lorem', bar: 'ipsum' }

console.log(foo) // "lorem"
console.log(bar) // "ipsum"
```

就像在数组上进行解构一样，你可以嵌套并结合进一步的解构：

``` javascript
var complicatedObj = {
    arrayProp: ['Zapp', { second: 'Brannigan' }]
}

var {
    arrayProp: [first, { second }]
} = complicatedObj

console.log(first) // "Zapp"
console.log(second) // "Brannigan"
```

当你对未定义的属性进行解构时，你会得到 `undefined`：

``` javascript
var { missing } = {}
console.log(missing) // undefined
```

你应该知道的一个潜在的小问题，就是当你使用一个对象上解构给变量赋值，而不是将它们声明（当没有 `var` 、`let` 或 `const`）：

``` javascript
{ blowUp } = { blowUp: 10 }
// Syntax error
```

发生这种情况是因为引擎试图将表达式解析为块语句（例如，`{ console }` 是有效的块语句）。解决方案是将模式或整个表达式包装在括号中：

``` javascript
({ safe }) = {}
({ andSound } = {})
// No errors
```

### 解构值不是对象或数组

当你尝试使用解构 `null` 或者时 `undefined` ，你会得到一个类型错误：

``` javascript
var [blowUp] = null
// TypeError: null has no properties
```

但是，您可以对其他基本类型（如布尔值，数字和字符串）进行解构，并获得 `undefined`：

``` javascript
var [wtf] = NaN
console.log(wtf)
// undefined
```

这可能会出乎意料，但经过进一步检查后，结果很简单。当一个值被解构时，它首先[使用抽象操作 ToObject 被 转换为一个对象](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-9.1.9)。大多数类型可以转换为一个对象，但 `null` 与 `undefined` 可能不是。

### 默认值

你还可以提供何时未解释您正在解构的属性的默认值：

``` javascript
var [missing = true] = []
console.log(missing) // true

var { x = 3 } = {}
console.log(x) // 3
```

### 延伸阅读

[Destructuring Assignment in ECMAScript 6](http://fitzgeraldnick.com/2013/08/15/destructuring-assignment-in-es6.html)
