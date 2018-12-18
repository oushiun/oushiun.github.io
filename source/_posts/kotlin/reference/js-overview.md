---
title: 使用 Kotlin 进行 JavaScript 开发

tags:
 - Kotlin
 - JavaScript
categories:
 - 后端
 - Kotlin
 - 参考
 - 概述
date: 2018-05-15 10:47:22
thumbnail: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 提供了 JavaScript 作为目标平台的能力。它通过将 Kotlin 转换为 JavaScript 来实现。目前的实现目标是 ECMAScript 5.1，但也有最终目标为 ECMAScript 2015 的计划。

当你选择 JavaScript 目标时，作为项目一部分的任何 Kotlin 代码以及 Kotlin 附带的标准库都会转换为 JavaScript。然而，这不包括使用的 JDK 和任何 JVM 或 Java 框架或库。任何不是 Kotlin 的文件会在编译期间忽略掉。

Kotlin 编译器努力遵循以下目标：

*   提供最佳大小的输出
*   提供可读的 JavaScript 输出
*   提供与现有模块系统的互操作性
*   在标准库中提供相同的功能，无论是 JavaScript 还是 JVM 目标（尽最大可能程度）。

<!-- more -->

## 如何使用

你可能希望在以下情景中将 Kotlin 编译为 JavaScript：

*   创建面向客户端 JavaScript 的 Kotlin 代码

    *   **与 DOM 元素交互**。Kotlin 提供了一系列静态类型的接口来与文档对象模型（Document Object Model）交互，允许创建和更新 DOM 元素。

    *   **与图形如 WebGL 交互**。你可以使用 Kotlin 在网页上用 WebGL 创建图形元素。

*   创建面向服务器端 JavaScript 的 Kotlin 代码

    *   **使用服务器端技术**。你可以使用 Kotlin 与服务器端 JavaScript（如 Node.js）进行交互

Kotlin 可以与现有的第三方库和框架（如 jQuery 或 ReactJS）一起使用。要使用强类型
API 访问第三方框架，可以使用 [ts2kt](https://github.com/kotlin/ts2kt) 工具将 TypeScript 定义从 [Definitely Typed](http://definitelytyped.org/)
类型定义仓库转换为 Kotlin。或者，你可以使用<!--
-->[动态类型](dynamic-type.html)访问任何框架，而无需强类型。

JetBrains 特地为 React 社区开发并维护了几个工具：[React bindings](https://github.com/JetBrains/kotlin-wrappers) 以及 [Create React Kotlin App](https://github.com/JetBrains/create-react-kotlin-app)。后者可以帮你开始使用 Kotlin 构建 React 应用程序而无需构建配置。

Kotlin 兼容 CommonJS、AMD 和 UMD，直截了当[与不同的模块系统交互](/docs/tutorials/javascript/working-with-modules/working-with-modules.html)。

## Kotlin 转 JavaScript 入门

要了解如何开始使用 JavaScript 平台的 Kotlin，请参考其[教程](/docs/tutorials/javascript/kotlin-to-javascript/kotlin-to-javascript.html)。
