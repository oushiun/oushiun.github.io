---
title: 动态类型

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - JavaScript
date: 2018-05-22 11:24:14
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

> 在面向 JVM 平台的代码中不支持动态类型

<!-- more -->

作为一种静态类型的语言，Kotlin 仍然需要与无类型或松散类型的环境（例如 JavaScript 生态系统）进行互操作。为了方便这些使用场景，语言中有 `dynamic` 类型可用：

```kotlin
val dyn: dynamic = ……
```

`dynamic` 类型基本上关闭了 Kotlin 的类型检查系统：

*   该类型的值可以赋值给任何变量或作为参数传递到任何位置；
*   任何值都可以赋值给 `dynamic` 类型的变量，或者传递给一个接受 `dynamic` 作为参数的函数；
*   `null`-检查对这些值是禁用的。

`dynamic` 最特别的特性是，我们可以对 `dynamic` 变量调用**任何**属性或以任意参数调用**任何**函数：

```kotlin
dyn.whatever(1, "foo", dyn) // “whatever”在任何地方都没有定义
dyn.whatever(*arrayOf(1, 2, 3))
```

在 JavaScript 平台上，该代码将按照原样编译：在生成的 JavaScript 代码中，Kotlin 中的 `dyn.whatever(1)` 变为 `dyn.whatever(1)`。

当在 `dynamic` 类型的值上调用 Kotlin 写的函数时，请记住由 Kotlin 到 JavaScript 编译器执行的名字修饰。你可能需要使用 [@JsName 注解](js-to-kotlin-interop.html#jsname-注解)为要调用的函数分配明确的名称。

动态调用总是返回 `dynamic` 作为结果，所以我们可以自由地这样链接调用：

```kotlin
dyn.foo().bar.baz()
```

当我们把一个 lambda 表达式传给一个动态调用时，它的所有参数默认都是 `dynamic` 类型的：

```kotlin
dyn.foo {
    x -> x.bar() // x 是 dynamic
}
```

使用 `dynamic` 类型值的表达式会按照原样转换为 JavaScript，并且不使用 Kotlin 运算符约定。支持以下运算符：

*   二元：`+`、 `-`、 `*`、 `/`、 `%`、 `>`、 `<`、 `>=`、 `<=`、 `==`、 `!=`、 `===`、 `!==`、 `&&`、 `||`
*   一元
    *   前置：`-`、 `+`、 `!`
    *   前置及后置：`++`、 `--`
*   赋值：`+=`、 `-=`、 `*=`、 `/=`、 `%=`
*   索引访问：
    *   读：`d[a]`，多于一个参数会出错
    *   写：`d[a1] = a2`，`[]` 中有多于一个参数会出错

`in`、 `!in` 以及 `..` 操作对于 `dynamic` 类型的值是禁用的。

更多技术说明请参见[规范文档](https://github.com/JetBrains/kotlin/blob/master/spec-docs/dynamic-types.md)。
