---
title: 协程

toc: true
tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 函数
date: 2018-05-17 20:35:52
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

> 在 Kotlin 1.1+ 中协程是*实验性的*。详见[下文](#协程的实验性状态)

一些 API 启动长时间运行的操作（例如网络 IO、文件 IO、CPU 或 GPU 密集型任务等），并要求调用者阻塞直到它们完成。协程提供了一种避免阻塞线程并用更廉价、更可控的操作替代线程阻塞的方法：协程*挂起*。

协程通过将复杂性放入库来简化异步编程。程序的逻辑可以在协程中*顺序*地表达，而底层库会为我们解决其异步性。该库可以将用户代码的相关部分包装为回调、订阅相关事件、在不同线程（甚至不同机器！）上调度执行，而代码则保持如同顺序执行一样简单。

许多在其他语言中可用的异步机制可以使用 Kotlin 协程实现为库。这包括源于 C# 和 ECMAScript 的 [`async`/`await`](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#composing-suspending-functions)、源于 Go 的 [管道](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#channels) 和 [`select`](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#select-expression) 以及源于 C# 和 Python [生成器/*yield*](#kotlin-coroutines-中的生成器-API)。关于提供这些结构的库请参见其[下文](coroutines.html#标准-API)描述。

<!-- more -->

## 阻塞 vs 挂起

基本上，协程计算可以*被挂起*而无需*阻塞线程*。线程阻塞的代价通常是昂贵的，尤其在高负载时，因为只有相对少量线程实际可用，因此阻塞其中一个会导致一些重要的任务被延迟。

另一方面，协程挂起几乎是无代价的。不需要上下文切换或者 OS 的任何其他干预。最重要的是，挂起可以在很大程度上由用户库控制：作为库的作者，我们可以决定挂起时发生什么并根据需求优化/记日志/截获。

另一个区别是，协程不能在随机的指令中挂起，而只能在所谓的*挂起点*挂起，这会调用特别标记的函数。

## 挂起函数

当我们调用标记有特殊修饰符 `suspend` 的函数时，会发生挂起：

``` kotlin
suspend fun doSomething(foo: Foo): Bar {
    ……
}
```

这样的函数称为*挂起函数*，因为调用它们可能挂起协程（如果相关调用的结果已经可用，库可以决定继续进行而不挂起）。挂起函数能够以与普通函数相同的方式获取参数和返回值，但它们只能从协程、其他挂起函数以及内联到其中的函数字面值中调用。

事实上，要启动协程，必须至少有一个挂起函数，它通常是匿名的（即它是一个挂起 lambda 表达式）。让我们来看一个例子，一个简化的 `async()` 函数（源自 [`kotlinx.coroutines`](#kotlincoroutines-中的生成器-api) 库）：

``` kotlin
fun <T> async(block: suspend () -> T)
```

这里的 `async()` 是一个普通函数（不是挂起函数），但是它的 `block` 参数具有一个带 `suspend` 修饰符的函数类型： `suspend () -> T`。所以，当我们将一个 lambda 表达式传给 `async()` 时，它会是*挂起 lambda 表达式*，于是我们可以从中调用挂起函数：

``` kotlin
async {
    doSomething(foo)
    ……
}
```

> **注意：**目前挂起函数类型不能用作超类型，并且目前不支持匿名挂起函数。

继续该类比，`await()` 可以是一个挂起函数（因此也可以在一个 `async {}` 块中调用），该函数挂起一个协程，直到一些计算完成并返回其结果：

``` kotlin
async {
    ……
    val result = computation.await()
    ……
}
```

更多关于 `async/await` 函数实际在 `kotlinx.coroutines` 中如何工作的信息可以在[这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#composing-suspending-functions)找到。

请注意，挂起函数 `await()` 与 `doSomething()` 不能在没有内联到挂起函数体的函数字面值以及像 `main()` 这样的普通函数中调用：

``` kotlin
fun main(args: Array<String>) {
    doSomething() // 错误：挂起函数从非协程上下文调用

    async {
        ...
        computations.forEach { // `forEach` 是一个内联函数，该 lambda 表达式是内联的
            it.await() // OK
}

        thread { // `thread` 不是内联函数，所以该 lambda 表达式并非内联的
            doSomething() // 错误
        }
    }
}
```

还要注意的是，挂起函数可以是虚拟的，当覆盖它们时，必须指定 `suspend` 修饰符：

``` kotlin
interface Base {
    suspend fun foo()
}

class Derived: Base {
    override suspend fun foo() { …… }
}
```

### `@RestrictsSuspension` 注解

扩展函数（和 lambda 表达式）也可以标记为 `suspend`，就像普通的一样。这允许创建 [DSL](type-safe-builders.html) 及其他用户可扩展的 API。在某些情况下，库作者需要阻止用户添加*新方式*来挂起协程。

为了实现这一点，可以使用 [`@RestrictsSuspension`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-restricts-suspension/index.html) 注解。当接收者类/接口 `R` 用它标注时，所有挂起扩展都需要委托给 `R` 的成员或其它委托给它的扩展。由于扩展不能无限相互委托（程序不会终止），这保证所有挂起都通过调用 `R` 的成员发生，库的作者就可以完全控制了。

这在*少数*情况是需要的，当每次挂起在库中以特殊方式处理时。例如，当通过 [`buildSequence()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-sequence.html) 函数实现[下文](#kotlincoroutines-中的生成器-api)所述的生成器时，我们需要确保在协程中的任何挂起调用最终调用 `yield()` 或 `yieldAll()` 而不是任何其他函数。这就是为什么 [`SequenceBuilder`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-sequence-builder/index.html) 用 `@RestrictsSuspension` 注解：

``` kotlin
@RestrictsSuspension
public abstract class SequenceBuilder<in T> {
    ……
}
```

参见其 [Github 上](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/coroutines/experimental/SequenceBuilder.kt) 的源代码。

## 协程的内部机制

我们不是在这里给出一个关于协程如何工作的完整解释，然而粗略地认识发生了什么是相当重要的。

协程完全通过编译技术实现（不需要来自 VM 或 OS 端的支持），挂起通过代码来生效。基本上，每个挂起函数（优化可能适用，但我们不在这里讨论）都转换为状态机，其中的状态对应于挂起调用。刚好在挂起前，下一状态与相关局部变量等一起存储在编译器生成的类的字段中。在恢复该协程时，恢复局部变量并且状态机从刚好挂起之后的状态进行。

挂起的协程可以作为保持其挂起状态与局部变量的对象来存储和传递。这种对象的类型是 `Continuation`，而这里描述的整个代码转换对应于经典的[延续性传递风格（Continuation-passing style）](https://en.wikipedia.org/wiki/Continuation-passing_style)。因此，挂起函数有一个 `Continuation` 类型的额外参数作为高级选项。

关于协程工作原理的更多细节可以在[这个设计文档](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)中找到。在其他语言（如 C# 或者 ECMAScript 2016）中的 async/await 的类似描述与此相关，虽然它们实现的语言功能可能不像 Kotlin 协程这样通用。

## 协程的实验性状态

协程的设计是[实验性的](compatibility.html#实验性的功能)，这意味着它会在即将发布的版本中更改。当在 Kotlin 1.1+ 中编译协程时，默认情况下会报一个警告：_“协程”功能是实验性的_。要移出该警告，你需要指定 [opt-in 标志](/docs/diagnostics/experimental-coroutines.html)。

由于其实验性状态，标准库中协程相关的 API 放在 `kotlin.coroutines.experimental` 包下。当设计完成并且实验性状态解除时，最终的 API 会移动到 `kotlin.coroutines`，并且实验包会被保留（可能在一个单独的构件中）以实现向后兼容。

**重要注意事项**：我们建议库作者遵循相同惯例：给暴露基于协程 API 的包添加“experimental”后缀（如 `com.example.experimental`），以使你的库保持二进制兼容。当最终 API 发布时，请按照下列步骤操作：

*   将所有 API 复制到 `com.example`（没有 experimental 后缀），
*   保持实验包的向后兼容性。

这将最小化你的用户的迁移问题。

## 标准 API

协程有三个主要组成部分：

*   语言支持（即如上所述的挂起功能）；
*   Kotlin 标准库中的底层核心 API；
*   可以直接在用户代码中使用的高级 API。

### 底层 API：`kotlin.coroutines`

底层 API 相对较小，并且除了创建更高级的库之外，不应该使用它。 它由两个主要包组成：

*   [`kotlin.coroutines.experimental`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/index.html) 带有主要类型与下述原语：
    *   [`createCoroutine()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/create-coroutine.html)，
    *   [`startCoroutine()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/start-coroutine.html)，
    *   [`suspendCoroutine()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/suspend-coroutine.html)；
*   [`kotlin.coroutines.experimental.intrinsics`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental.intrinsics/index.html) 带有甚至更底层的内在函数如 [`suspendCoroutineOrReturn`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental.intrinsics/suspend-coroutine-or-return.html)。

关于这些 API 用法的更多细节可以在[这里](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)找到。

### `kotlin.coroutines` 中的生成器 API

`kotlin.coroutines.experimental` 中仅有的“应用程序级”函数是

*   [`buildSequence()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-sequence.html)
*   [`buildIterator()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-iterator.html)

这些包含在 `kotlin-stdlib` 中因为他们与序列相关。这些函数（我们可以仅限于这里的 `buildSequence()`）实现了 _生成器_ ，即提供一种廉价构建惰性序列的方法：

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
    val fibonacciSeq = buildSequence {
        var a = 0
        var b = 1

        yield(1)

        while (true) {
            yield(a + b)

            val tmp = a + b
            a = b
            b = tmp
        }
    }
//sampleEnd

    // 输出前五个斐波纳契数字
    println(fibonacciSeq.take(8).toList())
}
```

这通过创建一个协程生成一个惰性的、潜在无限的斐波那契数列，该协程通过调用 `yield()` 函数来产生连续的斐波纳契数。当在这样的序列的迭代器上迭代每一步，都会执行生成下一个数的协程的另一部分。因此，我们可以从该序列中取出任何有限的数字列表，例如 `fibonacciSeq.take(8).toList()` 结果是 `[1, 1, 2, 3, 5, 8, 13, 21]`。协程足够廉价使这很实用。

为了演示这样一个序列的真正惰性，让我们在调用 `buildSequence()` 内部输出一些调试信息：

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
    val lazySeq = buildSequence {
        print("START ")
        for (i in 1..5) {
            yield(i)
            print("STEP ")
        }
        print("END")
    }

    // 输出序列的前三个元素
    lazySeq.take(3).forEach { print("$it ") }
//sampleEnd
}
```

运行上述代码会输出前三个元素。这些数字与生成循环中的 `STEP` 相交叉。这意味着计算确实是惰性的。要输出 `1`，我们只执行到第一个 `yield(i)`，并且过程中会输出 `START`。然后，输出 `2`，我们需要继续下一个 `yield(i)`，并会输出 `STEP`。`3` 也一样。永远不会输出再下一个 `STEP`（以及`END`），因为我们再也没有请求序列的后续元素。

为了一次产生值的集合（或序列），可以使用 `yieldAll()` 函数：

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
    val lazySeq = buildSequence {
        yield(0)
        yieldAll(1..10)
    }

    lazySeq.forEach { print("$it ") }
//sampleEnd
}
```

`buildIterator()` 的工作方式类似于 `buildSequence()`，但返回一个惰性迭代器。

可以通过为 `SequenceBuilder` 类写挂起扩展（带有[上文](#RestrictsSuspension-注解)描述的 `@RestrictsSuspension` 注解）来为 `buildSequence()` 添加自定义生产逻辑（custom yielding logic）：

``` kotlin
import kotlin.coroutines.experimental.*

//sampleStart
suspend fun SequenceBuilder<Int>.yieldIfOdd(x: Int) {
    if (x % 2 != 0) yield(x)
}

val lazySeq = buildSequence {
    for (i in 1..10) yieldIfOdd(i)
}
//sampleEnd

fun main(args: Array<String>) {
    lazySeq.forEach { print("$it ") }
}
```

### 其他高级 API：`kotlinx.coroutines`

只有与协程相关的核心 API 可以从 Kotlin 标准库获得。这主要包括所有基于协程的库可能使用的核心原语和接口。

大多数基于协程的应用程序级 API 都作为单独的库发布：[`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines)。这个库覆盖了

*   使用 `kotlinx-coroutines-core` 的平台无关异步编程：
    *   此模块包括支持 `select` 和其他便利原语的类似 Go 的管道，
    *   这个库的综合指南[在这里](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md)；
*   基于 JDK 8 中的 `CompletableFuture` 的 API：`kotlinx-coroutines-jdk8`；
*   基于 JDK 7 及更高版本 API 的非阻塞 IO（NIO）：`kotlinx-coroutines-nio`；
*   支持 Swing (`kotlinx-coroutines-swing`) 和 JavaFx (`kotlinx-coroutines-javafx`)；
*   支持 RxJava：`kotlinx-coroutines-rx`。

这些库既作为使通用任务易用的便利的 API，也作为如何构建基于协程的库的端到端示例。
