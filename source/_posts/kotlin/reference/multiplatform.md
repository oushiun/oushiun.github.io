---
title: 多平台项目

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 其他
date: 2018-05-22 10:21:57
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

> 多平台项目是 Kotlin 1.2 中的一个新的实验性功能。本文档中所描述的全部语言与工具功能都可能在未来的 Kotlin 版本中发生变化。

Kotlin 多平台项目允许你将相同的代码编译到多个目标平台。目前支持的目标平台为 JVM 与 JS，即将增加 Native。

<!-- more -->

### 多平台项目结构

多平台项目由三种类型的模块组成：

*   _公共_ 模块包含平台无关代码以及不含实现的平台相关 API 声明。这些声明允许公共代码依赖于平台相关实现。
*   _平台_ 模块包含公共模块中的平台相关声明针对指定平台的实现以及其他平台相关代码。一个平台模块始终是单个公共模块的一个实现。
*   常规模块。这类模块针对指定的平台，并且既可以成为平台模块的依赖也可以依赖于平台模块。

公共模块只能依赖其他公共模块与库，包括 Kotlin 标准库的公共版（`kotlin-stdlib-common`）。公共模块只包含 Kotlin 代码，而不包含任何其他语言代码。

平台模块可以依赖在指定平台上可用的任何模块与库（包括对于 Kotlin/JVM 平台的 Java 库与 Kotlin/JS 平台的 JS 库）。针对 Kotlin/JVM 平台的平台模块还可以包含 Java 以及其他 JVM 语言的代码。

编译一个公共模块会生成一个特殊的 _元数据_ 文件，其中包含模块中的所有声明。编译一个平台模块，会为平台模块中的代码以及它所实现的公共模块代码生成平台相关代码（JVM 字节码或者 JS 源代码）。

因此，每个多平台库需要分发为一组构件—— 一个包含用于公共代码的元数据的公共 .jar，以及包含用于每个平台的编译后的实现代码的多个平台相关 .jar。

### 设置多平台项目

截止到 Kotlin 1.2，多平台项目必须用 Gradle 构建；暂不支持其他构建系统。If you work with a multiplatform project in IDE, make sure that `Delegate IDE build/run actions to gradle` option is enabled and `Gradle Test Runner` is set for `Run tests using` option. Both options may be found here: _Settings > Build, execution, Deployment > Build Tools > Gradle > Runner_

要在 IDE 中创建一个新的多平台项目，请在“New Project”对话框中选择“Kotlin”下的“Kotlin (Multiplatform)”选项。这会创建一个具有三个模块的项目，一个公共项目以及分别用于 JVM 与 JS 平台的两个平台项目。要添加额外的模块，请在“New Module”对话框中选择“Gradle”下的“Kotlin (Multiplatform)”系列选项之一。

如需手动配置项目，请用以下步骤：

*   将 Kotlin Gradle 插件添加到构建脚本的类路径中：`classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"`。
*   将 `kotlin-platform-common` 插件应用到公共模块
*   将 `kotlin-stdlib-common` 依赖添加到公共模块中
*   将 `kotlin-platform-jvm`、 `kotlin-platform-android` 与 `kotlin-platform-js` 插件分别应用到 JVM、Android 与 JS 平台模块
*   将平台模块 `expectedBy` 作用域中添加到到公共模块的依赖

以下示例演示了一个使用 Kotlin 1.2 的公共模块的完整的 `build.gradle` 文件：

``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin-platform-common'

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-common:$kotlin_version"
    testCompile "org.jetbrains.kotlin:kotlin-test-common:$kotlin_version"
}
```

而下述示例展示了一个用于 JVM 平台模块的完整的 `build.gradle`。请特别注意其 `dependencies` 块中的 `expectedBy` 行：

``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin-platform-jvm'

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    expectedBy project(":")
    testCompile "junit:junit:4.12"
    testCompile "org.jetbrains.kotlin:kotlin-test-junit:$kotlin_version"
    testCompile "org.jetbrains.kotlin:kotlin-test:$kotlin_version"
}
```

### 平台相关声明

Kotlin 多平台代码的关键潜能之一就是公共代码依赖平台相关声明的一种方式。在其他语言中，这通常可以通过在公共代码中构建一组接口并在平台相关模块中实现这些接口来完成。不过，当有其中一个平台的库实现了所需功能，并且希望直接使用该库的 API 而无需额外包装时，这种方式并不理想。另外，它需要公共声明表示为接口，而这并不能覆盖所有可能场景。

作为替代方案，Kotlin 提供了一种 _预期与实际声明_ 机制。通过这种机制，公共模块中可定义 _预期声明_，而平台模块中可提供与预期声明相对应的 _实际声明_。为了解其工作原理，我们先看一个示例。这段代码是公共模块的一部分：

``` kotlin
package org.jetbrains.foo

expect class Foo(bar: String) {
    fun frob()
}

fun main(args: Array<String>) {
    Foo("Hello").frob()
}
```

而这是相应的 JVM 模块：

``` kotlin
package org.jetbrains.foo

actual class Foo actual constructor(val bar: String) {
    actual fun frob() {
        println("Frobbing the $bar")
    }
}
```

这阐明了几个重点：

*   公共模块中的预期声明与相应的实际声明总是具有完全相同的完整限定名。
*   预期声明标有 `expect` 关键字；实际声明标有 `actual` 关键字。
*   与预期声明的任何部分相匹配的所有实际声明都需要标记为 `actual`。
*   预期声明决不包含任何实现代码。

请注意，预期声明并不仅限于接口与接口成员。在本例中，预期的类有一个构造函数，可以直接在公共代码中创建该类。也可以将 `expect` 修饰符应用于其他声明，包括顶层声明与注解：

``` kotlin
// 公共
expect fun formatString(source: String, vararg args: Any): String

expect annotation class Test

// JVM 平台
actual fun formatString(source: String, vararg args: Any) =
    String.format(source, args)

actual typealias Test = org.junit.Test
```

编译器确保每个预期声明在实现相应公共模块的所有平台模块中都有实际声明，缺失任何实际声明都会报错。
IDE 提供了帮你创建所缺实际声明的工具。

如果你有一个平台相关的库，并希望在公共模块中使用，同时为另一平台提供自己的实现，那么你可以提供一个已有类的类型别名作为实际声明：

``` kotlin
expect class AtomicRef<V>(value: V) {
  fun get(): V
  fun set(value: V)
  fun getAndSet(value: V): V
  fun compareAndSet(expect: V, update: V): Boolean
}

actual typealias AtomicRef<V> = java.util.concurrent.atomic.AtomicReference<V>
```

### 多平台测试

可以在公共项目中编写测试，这样就可以在每个平台中编译与运行了。
`kotlin.test` 包中提供了 4 个注解用于标记公共代码中的测试：`@Test`、 `@Ignore`、 `@BeforeTest` 以及 `@AfterTest`.
在 JVM 平台中这些注解会映射到相应 JUnit 4 注解，而在 JS 中自 1.1.4 起它们也已可用于支持 JS 单元测试。

为了使用它们，你需要将依赖 `kotlin-test-annotations-common` 添加到你的公共模块，将依赖 `kotlin-test-junit` 添加到你的 JVM 模块，并且将依赖 `kotlin-test-js` 添加到 JS 模块。
