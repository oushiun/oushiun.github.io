---
title: This 表达式
toc: true
tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 其他
date: 2018-05-22 09:47:22
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

为了表示当前的 _接收者_ 我们使用 _this_ 表达式：

*   在[类](classes.html#继承)的成员中，_this_ 指的是该类的当前对象。
*   在[扩展函数](extensions.html)或者[带接收者的函数字面值](lambdas.html#带接收者的函数字面值)中，_this_ 表示在点左侧传递的 _接收者_ 参数。

如果 _this_ 没有限定符，它指的是最内层的包含它的作用域。要引用其他作用域中的 _this_，请使用 _标签限定符_：

<!-- more -->

### 限定的 _this_

要访问来自外部作用域的*this*（一个[类](classes.html) 或者[扩展函数](extensions.html)，或者带标签的[带接收者的函数字面值](lambdas.html#带接收者的函数字面值)）我们使用`this@label`，其中 `@label` 是一个<!--
-->代指 _this_ 来源的标签：

``` kotlin
class A { // 隐式标签 @A
    inner class B { // 隐式标签 @B
        fun Int.foo() { // 隐式标签 @foo
            val a = this@A // A 的 this
            val b = this@B // B 的 this

            val c = this // foo() 的接收者，一个 Int
            val c1 = this@foo // foo() 的接收者，一个 Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit 的接收者
            }


            val funLit2 = { s: String ->
                // foo() 的接收者，因为它包含的 lambda 表达式
                // 没有任何接收者
                val d1 = this
            }
        }
    }
}
```
