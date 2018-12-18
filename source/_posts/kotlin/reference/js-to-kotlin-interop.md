---
title: JavaScript 中调用 Kotlin

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - JavaScript
date: 2018-05-22 11:27:24
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 编译器生成正常的 JavaScript 类，可以在 JavaScript 代码中自由地使用的函数和属性。不过，你应该记住一些微妙的事情。

<!-- more -->

## 用独立的 JavaScript 隔离声明

为了防止损坏全局对象，Kotlin 创建一个包含当前模块中所有 Kotlin 声明的对象。所以如果你把模块命名为 `myModule`，那么所有的声明都可以通过 `myModule` 对象在 JavaScript 中可用。例如：

``` kotlin
fun foo() = "Hello"
```

可以在 JavaScript 中这样调用：

``` javascript
alert(myModule.foo())
```

这不适用于当你将 Kotlin 模块编译为 JavaScript 模块时（关于这点的详细信息请参见 [JavaScript 模块](js-modules.html)）。在这种情况下，不会有一个包装对象，而是将声明作为相应类型的 JavaScript 模块对外暴露。例如，对于 CommonJS 的场景，你应该写：

``` javascript
alert(require('myModule').foo())
```

## 包结构

Kotlin 将其包结构暴露给 JavaScript，因此除非你在根包中定义声明，否则必须在 JavaScript 中使用完整限定的名称。例如：

``` kotlin
package my.qualified.packagename

fun foo() = "Hello"
```

可以在 JavaScript 中这样调用：

``` javascript
alert(myModule.my.qualified.packagename.foo())
```

### `@JsName` 注解

在某些情况下（例如为了支持重载），Kotlin 编译器会修饰（mangle） JavaScript 代码中生成的函数和属性的名称。要控制生成的名称，可以使用 `@JsName` 注解：

``` kotlin
// 模块“kjs”
class Person(val name: String) {
    fun hello() {
        println("Hello $name!")
    }

    @JsName("helloWithGreeting")
    fun hello(greeting: String) {
        println("$greeting $name!")
    }
}
```

现在，你可以通过以下方式在 JavaScript 中使用这个类：

``` javascript
var person = new kjs.Person('Dmitry') // 引用到模块“kjs”
person.hello() // 输出“Hello Dmitry!”
person.helloWithGreeting('Servus') // 输出“Servus Dmitry!”
```

如果我们没有指定 `@JsName` 注解，相应函数的名称会包含从函数签名计算而来的后缀，例如 `hello_61zpoe$`。

请注意，Kotlin 编译器不会对 `external` 声明应用这种修饰，因此你不必在其上使用 `@JsName`。 值得注意的另一个例子是从外部类继承的非外部类。在这种情况下，任何被覆盖的函数也不会被修饰。

`@JsName` 的参数需要是一个常量字符串字面值，该字面值是一个有效的标识符。任何尝试将非标识符字符串传递给 `@JsName` 时，编译器都会报错。以下示例会产生编译期错误：

``` kotlin
@JsName("new C()")   // 此处出错
external fun newC()
```

## 在 JavaScript 中表示 Kotlin 类型

*   除了 `kotlin.Long` 的 Kotlin 数字类型映射到 JavaScript Number。
*   `kotlin.Char` 映射到 JavaScript Number 来表示字符代码。
*   Kotlin 在运行时无法区分数字类型（`kotlin.Long` 除外），即以下代码能够工作：

    ``` kotlin
    fun f() {
        val x: Int = 23
        val y: Any = x
        println(y as Float)
    }
    ```

*   Kotlin 保留了 `kotlin.Int`、 `kotlin.Byte`、 `kotlin.Short`、 `kotlin.Char` 和 `kotlin.Long` 的溢出语义。
*   JavaScript 中没有 64 位整数，所以 `kotlin.Long` 没有映射到任何 JavaScript 对象，它是由一个 Kotlin 类模拟的。
*   `kotlin.String` 映射到 JavaScript String。
*   `kotlin.Any` 映射到 JavaScript Object（即 `new Object()`、 `{}` 等）。
*   `kotlin.Array` 映射到 JavaScript Array。
*   Kotlin 集合（即 `List`、 `Set`、 `Map` 等）没有映射到任何特定的 JavaScript 类型。
*   `kotlin.Throwable` 映射到 JavaScript Error。
*   Kotlin 在 JavaScript 中保留了惰性对象初始化。
*   Kotlin 不会在 JavaScript 中实现顶层属性的惰性初始化。

自 1.1.50 版起，原生数组转换到 JavaScript 时采用 TypedArray：

*   `kotlin.ByteArray`、 `-.ShortArray`、 `-.IntArray`、 `-.FloatArray` 以及 `-.DoubleArray` 会相应地映射为 JavaScript 中的 Int8Array、 Int16Array、 Int32Array、 Float32Array 以及 Float64Array。
*   `kotlin.BooleanArray` 会映射为 JavaScript 中具有 `$type$ == "BooleanArray"` 属性的 Int8Array
*   `kotlin.CharArray` 会映射为 JavaScript 中具有 `$type$ == "CharArray"` 属性的 UInt16Array
*   `kotlin.LongArray` 会映射为 JavaScript 中具有 `$type$ == "LongArray"` 属性的 `kotlin.Long` 的数组。
