---
title: 使用 Kotlin 进行 Android 开发

tags:
 - Kotlin
 - Android
categories:
 - 后端
 - Kotlin
 - 概述
date: 2018-05-15 10:31:26
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 非常适合开发 Android 应用程序，将现代语言的所有优势带入 Android 平台而不会引入任何新的限制：

*   **兼容性**：Kotlin 与 JDK 6 完全兼容，保障了 Kotlin 应用程序可以在较旧的 Android 设备上运行而无任何问题。Kotlin 工具在 Android Studio 中会完全支持，并且兼容 Android 构建系统。
*   **性能**：由于非常相似的字节码结构，Kotlin 应用程序的运行速度与 Java 类似。随着 Kotlin 对内联函数的支持，使用 lambda 表达式的代码通常比用 Java 写的代码运行得更快。
*   **互操作性**：Kotlin 可与 Java 进行 100％ 的互操作，允许在 Kotlin 应用程序中使用所有现有的 Android 库。这包括注解处理，所以数据绑定和 Dagger 也是一样。
*   **占用**：Kotlin 具有非常紧凑的运行时库，可以通过使用 ProGuard 进一步减少。在[实际应用程序](https://blog.gouline.net/kotlin-production-tales-62b56057dc8a)中，Kotlin 运行时只增加几百个方法以及 .apk 文件不到 100K 大小。
*   **编译时长**：Kotlin 支持高效的增量编译，所以对于清理构建会有额外的开销，[增量构建通常与 Java 一样快或者更快](https://medium.com/keepsafe-engineering/kotlin-vs-java-compilation-speed-e6c174b39b5d)。
*   **学习曲线**：对于 Java 开发人员，Kotlin 入门很容易。包含在 Kotlin 插件中的自动 Java 到 Kotlin 的转换有助于迈出第一步。

<!-- more -->

### Kotlin 用于 Android 的案例学习

Kotlin 已被一些大公司成功采用，其中一些公司分享了他们的经验：

*   Pinterest 已经成功地[将 Kotlin 引入了他们的应用程序中](https://www.youtube.com/watch?v=mDpnc45WwlI)，每个月有 1 亿 5 千万人使用。
*   Basecamp 的 Android 应用程序是 [100％ Kotlin 代码](https://m.signalvnoise.com/how-we-made-basecamp-3s-android-app-100-kotlin-35e4e1c0ef12)，他们报告了程序员幸福的巨大差异，以及工作质量和速度的巨大改善。
*   Keepsafe 的 App Lock 应用程序也[已转换为 100％ Kotlin](https://medium.com/keepsafe-engineering/lessons-from-converting-an-app-to-100-kotlin-68984a05dcb6)，使源代码行数减少 30％、方法数减少 10％。

### 用于 Android 开发的工具

Kotlin 团队为 Android 开发提供了一套超越标准语言功能的工具：

*   [Kotlin Android 扩展](/docs/tutorials/android-plugin.html)是一个编译器扩展，可以让你摆脱代码中的 `findViewById()` 调用，并将其替换为合成的编译器生成的属性。
*   [Anko](http://github.com/kotlin/anko) 是一个提供围绕 Android API 的 Kotlin 友好的包装器的库，以及一个可以用 Kotlin 代码替换布局 .xml 文件的 DSL。

### 下一步

*   下载并安装 [Android Studio 3.0](https://developer.android.com/studio/index.html)，其中包含开箱即用的 Kotlin 支持。
*   按照 [Android 与 Kotlin 入门](/docs/tutorials/kotlin-android.html)教程创建你的第一个 Kotlin 应用程序。
*   关于更深入的介绍，请查看本站的[参考文档](index.html)。
*   另一个很好的资源是 [Kotlin for Android Developers](https://leanpub.com/kotlin-for-android-developers)，这本书会引导你逐步完成在 Kotlin 中创建真正的 Android 应用程序的过程。
*   检出 Google 的 [Kotlin 写的示例项目](https://developer.android.com/samples/index.html?language=kotlin)。
