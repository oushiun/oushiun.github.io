---
title: 增强的对象文本

tags:
 - ES6
categories:
 - 前端
 - JavaScript
 - ECMAScript 6
date: 2018-06-12 10:28:54
thumbnail: https://static.oushiun.com/blog/banner/enhanced-object-literals.jpg
---

使用对象文本可以做许多让人意想不到的事情！通过 ES6，我们可以把 ES5 中的 JSON 变得更加接近于一个类。

<!-- more -->

### 函数类属性的省略语法

用法：`{ method() {…} }`

``` javascript
const obj = {
    // Before
    foo: function() {
        return 'foo'
    },

    // After
    bar() {
        return 'bar'
    }
}
```

### 支持 proto 注入

开发者允许直接向一个对象字面量注入 _proto_，使其**直接成为指定类的一个实例**，而无须另外创建一个类来实现继承。

``` javascript
import { EventEmitter } from 'events'

const machine = {
    __proto__: new EventEmitter(),

    method() {
        /* …*/
    }
    // …
}

console.log(machine) //EventEmitter {}
console.log(machine instanceof EventEmitter) //true

machine.on('event', msg => console.log(`Received message: ${msg}`))
machine.emit('event', 'hello world')
// Received message: hello world

machine.method(/* …. */)
```

### 可动态计算的属性名

ES6 引入的新语法允许我们直接使用一个表达式来表达一个属性名用法：`{ [statement]: value }`

``` javascript
const prefix = 'ES6'

const obj = {
    [prefix + 'enhancedObject']: 'foo'
}
```

### 将属性名定义省略

有时候我们需要将一些已经被定义的变量(或常量)作为其他对象字面量的属性值进行返回或传入操作，而大多数情况下这些变量名和属性名都是相同的，我们可以对属性名定义进行省略。

``` javascript
const foo = 123
const bar = () => foo

const obj = {
    foo,
    bar
}

console.log(obj) //{ foo: 123, bar: [Function] }
```
