---
title: 使用 Kotlin 进行服务器端开发

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 概述
date: 2018-05-15 09:22:38
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

Kotlin 非常适合开发服务器端应用程序，允许编写简明且表现力强的代码，同时保持与现有基于 Java 的技术栈的完全兼容性以及平滑的学习曲线：

*   **表现力**：Kotlin 的革新式语言功能，例如支持[类型安全的构建器](type-safe-builders.html)和[委托属性](delegated-properties.html)，有助于构建强大而易于使用的抽象。
*   **可伸缩性**：Kotlin 对[协程](coroutines.html)的支持有助于构建服务器端应用程序，伸缩到适度的硬件要求以应对大量的客户端。
*   **互操作性**：Kotlin 与所有基于 Java 的框架完全兼容，可以让你保持熟悉的技术栈，同时获得更现代化语言的优势。
*   **迁移**：Kotlin 支持大型代码库从 Java 到 Kotlin 逐步迁移。你可以开始用 Kotlin 编写新代码，同时系统中较旧部分继续用 Java。
*   **工具**：除了很棒的 IDE 支持之外，Kotlin 还为 IntelliJ IDEA Ultimate 的插件提供了框架特定的工具（例如
    Spring）。
*   **学习曲线**：对于 Java 开发人员，Kotlin 入门很容易。包含在 Kotlin 插件中的自动 Java 到 Kotlin 的转换器有助于迈出第一步。

<!-- more -->

### 使用 Kotlin 进行服务器端开发的框架

*   [Spring](https://spring.io) 利用 Kotlin 的语言功能提供[更简洁的 API](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0)，从版本 5.0 开始。[在线项目生成器](https://start.spring.io/#!language=kotlin)允许用 Kotlin 快速生成一个新项目。

*   [Vert.x](http://vertx.io) 是在 JVM 上构建响应式 Web 应用程序的框架，为 Kotlin 提供了[专门支持](https://github.com/vert-x3/vertx-lang-kotlin)，包括[完整的文档](http://vertx.io/docs/vertx-core/kotlin/)。

*   [Ktor](https://github.com/kotlin/ktor) 是由 JetBrains 构建的 Kotlin 原生 Web 框架，利用协程实现高可伸缩性，并提供易于使用且合乎惯用法的 API。

*   [kotlinx.html](https://github.com/kotlin/kotlinx.html) 是可在 Web 应用程序中用于构建 HTML 的 DSL。它可以作为传统模板系统（如 JSP 和 FreeMarker）的替代品。

*   通过相应 Java 驱动程序进行持久化的可用选项包括直接 JDBC 访问、JPA 以及使用 NoSQL 数据库。对于 JPA，[kotlin-jpa 编译器插件](compiler-plugins.html#JPA-支持)使 Kotlin 编译的类适应框架的要求。

### 部署 Kotlin 服务器端应用程序

Kotlin 应用程序可以部署到支持 Java Web 应用程序的任何主机，包括 Amazon Web Services、
Google Cloud Platform 等。

要在 [Heroku](https://www.heroku.com) 上部署 Kotlin 应用程序，可以按照 [Heroku 官方教程](https://devcenter.heroku.com/articles/getting-started-with-kotlin)来做。

AWS Labs 提供了一个[示例项目](https://github.com/awslabs/serverless-photo-recognition)，展示了 Kotlin 编写 [AWS Lambda](https://aws.amazon.com/lambda/) 函数的使用。

### Kotlin 用于服务器端的用户

[Corda](https://www.corda.net/2017/01/10/kotlin/) 是一个开源的分布式分类帐平台，由各大银行提供支持，完全由 Kotlin 构建。

[JetBrains 账户](https://account.jetbrains.com/)，负责 JetBrains 整个许可证销售和验证过程的系统 100％ 由 Kotlin 编写，自 2015 年生产运行以来，一直没有重大问题。

### 下一步

*   [使用 Http Servlet 创建 Web 应用程序](/docs/tutorials/httpservlets.html) 及 [使用 Spring Boot 创建 RESTful Web 服务](/docs/tutorials/spring-boot-restful.html)教程将向你展示如何在 Kotlin 中构建和运行非常小的 Web 应用程序。
