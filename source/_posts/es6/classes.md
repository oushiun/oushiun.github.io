---
title: classes

tags:
 - ES6
categories:
 - 前端
 - JavaScript
 - ECMAScript 6
date: 2018-06-11 16:07:32
thumbnail: https://static.oushiun.com/blog/banner/classes.jpeg
---

### 简介

ECMAScript 6 引入的 JavaScript 类（class）是 JavaScript 现有的原型继承的语法糖。

javascript 传统做法是当生成一个对象实例，需要定义构造函数，然后通过 new 的方式完成。

``` javascript
function StdInfo() {
    this.name = 'job'
    this.age = 30
}

StdInfo.prototype.getNames = function() {
    console.log('name：' + this.name)
}
// 得到一个学员信息对象
var p = new StdInfo()
```

javacript 中只有对象，没有类。它是是基于原型的语言，原型对象是新对象的模板，它将自身的属性共享给新对象。这样的写法和传统面向对象语言差异很大，很容易让新手感到困惑。

<!-- more -->

### 定义类

到了 ES6 添加了类，作为对象的模板。通过 class 来定义一个类：

``` javascript
// 定义类
class StdInfo {
    constructor() {
        this.name = 'job'
        this.age = 30
    }
    //定义在类中的方法不需要添加function
    getNames() {
        console.log('name：' + this.name)
    }
}

// 使用new的方式得到一个实例对象
var p = new StdInfo()
```

上面的写法更加清晰、更像面向对象编程的语法，看起来也更容易理解。

定义的类只是语法糖，目的是让我们用更简洁明了的语法创建对象及处理相关的继承。

``` javascript
// 定义类
class StdInfo {
    //...
}
console.log(typeof StdInfo) //function

console.log(StdInfo === StdInfo.prototype.constructor) //true
```

从上面的测试中可以看出来，类的类型就是一个函数，是一个“特殊函数”，指向的是构造函数。

函数的定义方式有函数声明和函数表达式两种，类的定义方式也有两种，分别是：[类声明](#类声明) 和 [类表达式](#类表达式)。

### 类声明

类声明是定义类的一种方式，使用关键字 class，后面跟上类名，然后就是一对大括号。把这一类需要定义的方法放在大括号中。

``` javascript
// 定义类，可以省略constructor
class StdInfo {
    getNames() {
        console.log('name：' + this.name)
    }
}

// 定义类，加上constructor
class StdInfo {
    //使用new定义实例对象时，自动调用这个函数，传入参数
    constructor(name, age) {
        this.name = name
        this.age = age
    }

    getNames() {
        console.log('name：' + this.name)
    }
}

// 定义实例对象时，传入参数
var p = new StdInfo('job', 30)
```

_constructor_ 是一个默认方法，使用 new 来定义实例对象时，自动执行 _constructor_ 函数，传入所需要的参数,执行完 _constructor_ 后自动返回实例对象。

一个类中只能有一个 _constructor_ 函数，定义多个会报错。

_constructor_ 中的 `this` 指向新创建的实例对象，利用 `this` 往新创建的实例对象扩展属性。

在定义实例对象时，不需要在初始化阶段做一些事，可以不用显示的写 _constructor_ 函数。如果没有显式定义，一个空的 _constructor_ 方法会被默认添加，`constructor(){}`

### 类表达式

类表达式是定义类的另一种形式，类似于函数表达式，把一个函数作为值赋给变量。可以把定义的类赋值给一个变量，这时候变量就为类名。_class_ 关键字之后的类名可有可无，如果存在，则只能在类内部使用。

定义类 _class_ 后面有类名：

``` javascript
const People = class StdInfo {
    constructor() {
        console.log(StdInfo) //可以打印出值，是一个函数
    }
}

new People()
new StdInfo() //报错，StdInfo is not defined；
```

定义类 class 后面没有类名：

``` javascript
const People = class {
    constructor() {}
}

new People()
```

立即执行的类：

``` javascript
const p = new class {
    constructor(name, age) {
        console.log(name, age)
    }
}('job', 30)
```

立即执行的类，在类前要加上 **new**。`p` 为类的实例对象。

### 不存在变量提升

定义类不存在变量提升，只能先定义类后使用，跟函数声明有区别的。

``` javascript
//-----函数声明-------
// 定义前可以先使用，因为函数声明提升的缘故，调用合法。
func()
function func() {}

//-----定义类---------------
new StdInfo() // 报错，StdInfo is not defined
class StdInfo {}
```

### extends 继承

使用 `extends` 关键字实现类之间的继承。这比在 ES5 中使用继承要方便很多。

``` javascript
// 定义类父类
class Parent {
    constructor(name, age) {
        this.name = name
        this.age = age
    }

    speakSometing() {
        console.log('I can speek chinese')
    }
}
// 定义子类，继承父类
class Child extends Parent {
    coding() {
        console.log('coding javascript')
    }
}

var c = new Child()

// 可以调用父类的方法
c.speakSometing() // I can speek chinese
```

使用继承的方式，子类就拥有了父类的方法。

如果子类中有 _constructor_ 构造函数，则必须使用调用 `super`。

``` javascript
// 定义类父类
class Parent {
    constructor(name, age) {
        this.name = name
        this.age = age
    }

    speakSometing() {
        console.log('I can speek chinese')
    }
}
// 定义子类，继承父类
class Child extends Parent {
    constructor(name, age) {
        // 不调super()，则会报错  this is not defined
        // 必须调用super
        super(name, age)
    }

    coding() {
        console.log('coding javascript')
    }
}

var c = new Child('job', 30)

// 可以调用父类的方法
c.speakSometing() // I can speek chinese
```

子类必须在 _constructor_ 方法中调用 `super` 方法，否则新建实例时会报错(`this is not defined`)。这是因为子类没有自己的 `this` 对象，而是继承父类的 `this` 对象，然后对其进行加工。如果不调用 `super` 方法，子类就得不到 `this` 对象。

### 延伸阅读

[Classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
