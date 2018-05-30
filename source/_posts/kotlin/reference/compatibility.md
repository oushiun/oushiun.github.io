---
title: 兼容性

tags:
 - Kotlin
categories:
 - 后端
 - Kotlin
 - 参考
 - 参考
date: 2018-05-22 11:11:08
banner: https://static.oushiun.com/blog/banner/Kotlin.png
---

### 兼容性词汇表

兼容性意味着回答这个问题：对于给定的两个版本的 Kotlin（例如，1.2 和 1.1.5），为一个版本编写的代码可以与另一个版本一起使用吗？下面的列表解释了不同版本对的兼容模式。请注意，如果版本号较小（即使发布时间晚于版本号较大的版本）那么版本较旧。对于“旧版本”我们使用 OV，对于“新版本”使用 NV。

<!-- more -->

- **C**——完全兼容（Full **C**ompatibility）
  - 语言
    - 无语法改动（*除去 bug*\*）
    - 可能添加或删除新的警告/提示
  - API（`kotlin-stdlib-*`、 `kotlin-reflect-*`）
    - 无 API 改动
    - 可能添加/删除带有 `WARNING` 级的弃用项
  - 二进制（ABI）
    - 运行时：二进制可以互换使用
    - 编译：二进制可以互换使用
- **BCLA**——语言和 API 向后兼容（**B**ackward **C**ompatibility for the **L**anguage and **A**PI）
  - 语言
    - 可能会在 NV 中删除 OV 中已弃用的语法
    - 除此之外，OV 中可编译的所有代码都可以在 NV 中编译（除去 bug*）
    - 可能在 NV 中添加新语法
    - 在 NV 中可以提升 OV 的一些限制
    - 可能添加或删除新的警告/提示
  - API（`kotlin-stdlib-*`、 `kotlin-reflect-*`）
    - 可能添加新的 API
    - 可能添加/删除带有 `WARNING` 级的弃用项
    - `WARNING` 级的弃用项可能在 NV 中提升到 `ERROR` 级或者 `HIDDEN` 级
- **BCB**——二进制向后兼容（**B**ackward **C**ompatibility for **B**inaries）
  - 二进制（ABI）
    - 运行时：NV 的二进制可以在 OV 的二进制工作的任何地方使用
    - NV 编译器：针对 OV 二进制编译的代码可针对 NV 二进制编译
    - OV 编译器可能不接受 NV 二进制（例如，展示较新语言特性或 API 的二进制）
- **BC**——完全向后兼容（Full **B**ackward **C**ompatibility）
  - BC = BCLA & BCB
- **EXP**——实验性的功能（Experimental feature）
  - 参见[下文](#实验性的功能)
- **NO**——无兼容性保证（No compatibility guarantees）
  - 我们会尽力提供顺利的迁移，但不能给出任何保证
  - 为每个不兼容的子系统单独规划迁移

---

\* *除去 bugs* 无改动意味着如果发现一个重要的 bug（例如在编译器诊断或其他地方），修复它可能会引入一个破坏性改动，但我们总是非常小心对待这样的改动。

### Kotlin 发行版的兼容性保证

**JVM 平台的 Kotlin**：
  - 补丁版本更新（例如1.1.X）完全兼容；
  - 次版本更新（例如1.X）向后兼容。

| Kotlin    | 1.0   | 1.0.X | 1.1   | 1.1.X | ...   | 2.0   |
| --------: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1.0**   | -     | C     | BC    | BC    | ...   | ?     |
| **1.0.X** | C     | -     | BC    | BC    | ...   | ?     |
| **1.1**   | BC    | BC    | -     | C     | ...   | ?     |
| **1.1.X** | BC    | BC    | C     | -     | ...   | ?     |
| **...**   | ...   | ...   | ...   | ...   | ...   | ...   |
| **2.0**   | ?     | ?     | ?     | ?     | ...   | -     |

**JS 平台的 Kotlin**：从 Kotlin 1.1 开始，补丁版本和次版本更新为语言和 API 提供向后兼容性（BCLA），但没有 BCB。

| Kotlin    | 1.0.X | 1.1   | 1.1.X | ...   | 2.0   |
| --------: | :---: | :---: | :---: | :---: | :---: |
| **1.0.X** | -     | EXP   | EXP   | ...   | EXP   |
| **1.1**   | EXP   | -     | BCLA  | ...   | ?     |
| **1.1.X** | EXP   | BCLA  | -     | ...   | ?     |
| **...**   | ...   | ...   | ...   | ...   | ...   |
| **2.0**   | EXP   | ?     | ?     | ...   | -     |

**Kotlin Scripts**：补丁版本和次版本更新为语言和 API 提供向后兼容性（BCLA），但没有 BCB。

### 跨平台兼容性
 
Kotlin 可用于多个平台（JVM/Android、JavaScript 以及即将推出的本地平台）。每个平台都有自己的特殊性（例如 JavaScript 没有适当的整数），因此我们必须相应地调整语言。我们的目标是提供合理的代码可移植性，而不会牺牲太多。
  
每个平台都可能具有特定的语言扩展（例如 JVM 的平台类型和 JavaScript 的动态类型）或限制（例如 JVM 上与重载相关的限制），但核心语言保持不变。

标准库提供了在所有平台上可用的核心 API，我们努力使这些 API 在每个平台上以相同的方式工作。除此之外，标准库提供了平台相关的扩展（例如，JVM 的` java.io` 或 JavaScript 的 `js()`）以及一些可以统一调用但工作方式不同的 API（例如 JVM 和 JavaScript 的正则表达式）。

### 实验性的功能

实验性的功能，如 Kotlin 1.1 中的协程，可以从上面列出的兼容模式中豁免。这类功能需要选择性加入（opt-in）来使用才没有编译器警告。实验性的功能至少向后兼容补丁版本更新，但我们不保证任何次版本更新的兼容性（会尽可能提供迁移帮助）。

| Kotlin    | 1.1   | 1.1.X | 1.2   | 1.2.X |  |
| --------: | :---: | :---: | :---: | :---: |
| **1.1**   | -     | BC    | NO    | NO    |
| **1.1.X** | BC    | -     | NO    | NO    |
| **1.2**   | NO    | NO    | -     | BC    |
| **1.2.X** | NO    | NO    | BC    | -     |

### EAP 构建版

我们发布早期访问预览（Early Access Preview，EAP）构建版到特殊渠道，该社区的早期采用者可以试用它们并提供反馈。这样的构建不提供任何兼容性保证（尽管我们尽最大努力保持它们与发行版以及彼此之间的合理的兼容性）。这类构建版的质量预期也远低于发行版。Beta 测试版本也属于这一类别。

**重要注意事项**：通过 EAP 为 1.X（例如 1.1.0-eap-X）编译的所有二进制文件**会被编译器发行版版本拒绝**。我们不希望预发布版本编译的任何代码在稳定版本发布后保留。这不涉及补丁版本的 EAP（例如 1.1.3-eap-X），这些 EAP 产生具有稳定 ABI 的构建。

### 兼容性模式

当一个大团队迁移到一个新版本时，当一些开发人员已经更新、而其他人没有时，可能会在某个时候出现“不一致的状态”。为了防止前者编写和提交别人可能无法编译的代码，我们提供了以下命令行开关（在 IDE 以及 [Gradle](using-gradle.html#编译器选项)/[Maven](using-maven.html#指定编译器选项) 中也可用）：

- `-language-version X.Y`——Kotlin 语言版本 X.Y 的兼容性模式，对其后出现的所有语言功能报告错误。
- `-api-version X.Y`——Kotlin API 版本 X.Y 的兼容性模式，对使用来自 Kotlin 标准库（包括编译器生成的代码）的新版 API 的所有代码报告错误。

### 二进制兼容性警告

如果使用 NV Kotlin 编译器并在 classpath 中配有 OV 标准库或 OV 反射库，那么可能是项目配置错误的迹象。
为了防止编译期或运行时出现意外问题，我们建议要么将依赖关系更新到 NV，要么明确指定 API 版本/语言版本参数。
否则编译器会检测到某些东西可能出错，并报告警告。

例如，如果 OV = 1.0 且 NV = 1.1，你可能观察到以下警告之一：

```
Runtime JAR files in the classpath have the version 1.0, which is older than the API version 1.1. 
Consider using the runtime of version 1.1, or pass '-api-version 1.0' explicitly to restrict the 
available APIs to the runtime of version 1.0.
```

这意味着你针对版本 1.0 的标准库或反射库使用 Kotlin 编译器 1.1。这可以通过不同的方式处理：
* 如果你打算使用 1.1 标准库中的 API 或者依赖于这些 API 的语言特性，那么应将依赖关系升级到版本 1.1。
* 如果你想保持你的代码与 1.0 标准库兼容，你可以传参 `-api-version 1.0`。
* 如果你刚刚升级到 kotlin 1.1，但不能使用新的语言功能（例如，因为你的一些队友可能没有升级），你可以传参 `-language-version 1.0`，这会限制所有的 API 和语言功能到 1.0。

```
Runtime JAR files in the classpath should have the same version. These files were found in the classpath:
    kotlin-reflect.jar (version 1.0)
    kotlin-stdlib.jar (version 1.1)
Consider providing an explicit dependency on kotlin-reflect 1.1 to prevent strange errors
Some runtime JAR files in the classpath have an incompatible version. Consider removing them from the classpath
```

这意味着你对不同版本的库有依赖性，例如 1.1 标准库和 1.0 反射库。为了防止在运行时出现微妙的错误，我们建议你使用所有 Kotlin 库的相同版本。在本例中，请考虑对 1.1 反射库添加显式依赖关系。

```
Some JAR files in the classpath have the Kotlin Runtime library bundled into them. 
This may cause difficult to debug problems if there's a different version of the Kotlin Runtime library in the classpath. 
Consider removing these libraries from the classpath
```

这意味着在 classpath 中有一个库，它不是作为 Gradle/Maven 依赖项而依赖 Kotlin 标准库，而是与它分布在同一个构件中（即是*被捆绑的*）。这样的库可能会导致问题，因为标准构建工具不认为它是 Kotlin 标准库的实例，因此它不受依赖版本解析机制的限制，你可以在 classpath 找到同一个库的多个版本。请考虑联系这样的库的作者，并提出使用 Gradle/Maven 依赖取代的建议。