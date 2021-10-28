---
layout: default
title: 云研发架构
nav_order: 3
description: "云研发，是一种生于云上的闭环 + 代码化的软件开发方式。它可以让业务人员、开发人员、运营人员等在同一个云端共同协作、透明化地完成整个软件的生命周期（需求、设计、编码、构建、部署、运营），而非相互隔离，又或者是借助于多个软件才能完成工作。"
---

# 云研发架构 Water
{: .fs-9 .no_toc .text-delta }

## 目录
{: .no_toc .text-delta }

1. TOC
{:toc}

> Water 编码架构，即人类编程的产出不再是编程语言这一类型的字符串，编程时只是这种架构模式在 UI 上的呈现，其最终的存储的形式则是：一系列的代码的中间形式，如 AST、HIR、MIR 等。而代码不再被操作系统及文件系统束缚，它们可以交由更小粒度的架构因素控制，即函数。代码不再需要以文件的方式存储，系统的架构不再是按目录划分的分层架构。

## 引子 1：无代码与云研发

> 代码的复杂度，同力一样不会消失，也不会凭空产生，它总是从一种形式转为另一种形式。

2019 年，因为中台的火爆 + Serverless 的崛起，我写了一篇很长的文章在介绍如何设计《[无代码编程](https://www.phodal.com/blog/low-code-programming/)》。在这两年的时光里，出现了越来越多的所谓的『低代码平台』，但是它们并非是我所想的类型，缺少关键性的 DSL 抽象。不过呢，无代码系统架构的设计和本文的主题并没有太大的关系。但是呢，它们掀起了开发人员对于云端开发的浪潮。

 - 云设计。如 BeeArt 将大量的设计工作搬到了云端来完成。
 - 云 IDE。如 VSCode Remote、Eclipse Theia，它们可以将新的应用部署到远程开发。
 - 云开发环境。如 Nocalhost，它们可以解决存量系统的云端开发问题。
 - ……

不过，这里我们要举一个反面例子：『云主机』，这种远程开发模式可不是云研发。从本地 IDE 到云 IDE，本地设计软件上云等等的一系列效率工具的云迁移，它将使得我们在未来在云上（浏览器端）完成整个研发。而对于我们这些开发人员来说，重要的不是结果，而是如何去实现这样一个架构。

## 引子 2：代码架构与操作系统

> 分层架构就是建文件夹。

接着，我们来说一些更有意思的事情，代码存在的形式。在现今的软件开发中，我们以目录作为边界来设计分层架构，以文件作为代码的载体。而这种形式存在的一个主要原因是，我们开发时依赖于操作系统的存储：文件系统。对于文件系统而言，文件和树形目录的抽象逻辑概念便是人类所能接受的概念。

### 分层架构 vs 模块 vs 包

如果你熟悉 Java 语言的话，你会发现操作系统限制了 Java  语言的软件架构。一个主要的特征就是包，即目录即是包，`com.phodal.water` 对应了 `com/phodal/water`。而到了今天我们习惯了使用目录这种机制来进行包管理。如在最近几年开始流行开来的整洁架构，它是一种圆环型（or 洋葱式）架构，它就受限于目录的影响，使得系统最后的实现并不是那么直观。

(ps：由于篇幅所限，这里就不展示作更详细的介绍了。）

如果你对于整洁架构又或者是分层架构不是非常了解，那么你可以参考我之前相关的文章和开源项目：

 - 《整洁前端架构》：https://github.com/phodal/clean-frontend
 - Layer Architecture：https://github.com/phodal/layer-architecture

### 文件 vs 类

在 Unix/Linux 系统中，一切皆文件。

对于编程语言来说，我们常用的好习惯是：一个类只在一个文件里。尽管对于 Java 以外的语言来说，并非如此，但是这样实现易读、易维护。与此同时，我们也习惯于将相似的行为构建在同一个类中，即富血模型。因此，在那本开源电子书《[系统重构与迁移指南](https://github.com/phodal/migration)》，我一直在说服人们这样去做。

过去，我们一直局限于文件的规则。所以，让我们考虑一下问题，如果我们可以脱离操作系统呢？如果说，我们不带用文件来约束类，那么就不需要这一类规则。

## 引子 3：代码的中间表示

为了实现上面的一系列概念，我们需要重新设计整个系统，其中的一个关键部分就是：代码的中间表示。如果你熟悉编译原理的话，也就懂了，只有给人的时候，才会以编程语言的形式出现。

### 开发态

对于我们来说，只要我们看到的是易于阅读的代码，它是中文的、英文的，又或者是甲骨文都不重要。反正，我们写的代码是给自己看的，这些字符最后会经过层层转换为特殊的格式。

而在 IDE 与编辑器里，为了实现对于编程语言的高亮、智能感知等的支持，需要重新实现语言的类 AST 解析。如 VSCode 里的 LSP，Intellij IDEA 中的 PSI，Textmate/VSCode 中的 textmate 高亮语法等等。

所以，让我们思考一个问题，如果我们是以类 AST 的形式存储的话，那么我们就不需要实现这一类解析。它们只需要在展示的时候，将其转换为人类所熟悉的编程语言即可。

### 编译态/运行态

在和同事一起设计 Datum（原 Charj，更名为 Datum）语言的半年期间，我分析了市面上主流的一些语言的中间表示，如 Python 和 JVM 的  bytecode，Rust 的 MIR 等等。它们就是各种中间表示，但是相差并不是非常多，原理也是类似的。它们都是由上一步的 AST 转换而来的，以接近底层的方式重新设计。

这一个时候，还引发了一个更有意思的问题：如果某部分代码没有变更的话，那么我需要重新编译吗？既然某一个特定的部分没有办法，那么它就不会发生改变。

与此同时，一个更有意思的事情是，所有的修改只需要打个 patch 即可，应用再重新运行。

## 新代码架构：water

> 编程语言是写给人看的代码，写给机器运行的机器码。

基于上述的思想，我们可以设计新一代的代码架构：water。因为代码的形态已经不再重要了，用什么语言写也不再重要了，只需要一个统一的后端（编译器后端）即可。让我们先给出第一个版本的架构定义：

> Water 编码架构，即人类编程的产出不再是编程语言这一类型的字符串，编程时只是这种架构模式在 UI 上的呈现，其最终的存储的形式则是：一系列的代码的中间形式，如 AST、HIR、MIR 等。而代码不再被操作系统及文件系统束缚，它们可以交由更小粒度的架构因素控制，即函数。代码不再需要以文件的方式存储，系统的架构不再是按目录划分的分层架构。

从上述的定义来看，它具备以下的特性：

**语言无关**。因为存储的形态是 AST 形式的 DSL，所以开发人员可以用任意的语言开发。如 A 使用 Java 语言开发，B 使用 JavaScript 语言开发，它们在存储时将可以转换为统一的 AST。而 B 程序员在自己的编辑器上使用的是呈现语言，所以 B 看到的 A 写的代码可以是 JavaScript，而不再需要是 Java 语言。再理于 Java 程序员 A，A 使用的是 Java，它可以选择的呈现语言是 Java，所以哪怕一个 C 程序员选了 Golang，它也将看到的是 Java 的呈现。（PS：当然了，这只是理论上，还有诸多的问题有待解决。）
 
**细粒度权限管理**。现今的代码管理机制之下，对于开发者权限的管理是以代码库为单位的。但是在新的架构模式之下，它可以控制到函数的级别。主要是由于代码已经变为了数据，开发人员只编写模型和行为。它还能实现再更细的粒度上，结合我司大佬提出的 Typeflow 的思想，就可以控制到函数的粒度进行权限管理。即特定的开发人员，只拥有修改某一**特定业务代码集**权限。
 
**自动化架构**。经典的架构机制依赖于文件夹的定义，相似行为和功能的代码以文件夹（也称之为包）的形式统一。而当我们消除了架构中文件和文件夹的概念之后，每个模型及其行为以新的形式存在。试想一下在图数据库中，我们如何去表示数据及其关系，那么它就可以不断自动地优化架构，如基于聚类算法实现自动包定义。

**一切皆数据**。在 Unix/Linux 系统中，一切都是文件。而到了云时，一切都是数据。尽管，从某种意义上来说，文件也是数据的一种。代码本身也是数据。

其它一些有意思的内容：

**语言数据库**。当我们把编程语言转换为数据之后，我们面临地挑战有两个：数据的形式以及如何存储？即使在 5G 的场景之下，我们也需要考虑一下 4G 网络传输的问题，那么传输的介质可能是某种二进制形式的包，如 Android 里的 dex 包的形式，并带有其它丰富的信息。

**编辑态优先**。脱离了语言的限制之后，这种架构模式之下，我们还要考虑的一大因素是：编辑器/IDE 上的呈现与交互。我们需要实现快速其的编辑与反馈，因此编辑态优先。

还有其它一些有意思东西，我们可以想象一下。

### 超越 Serverless 

于是乎，我们能联想到另外一个有意思的概念是 Serverless。

>  Serverless 架构是指大量依赖第三方服务（也叫做后端即服务，即“BaaS”）或暂存容器中运行的自定义代码（函数即服务，即“FaaS”）的应用程序，函数是无服务器架构中抽象语言运行时的最小单位。在这种架构中，我们并不看重运行一个函数需要多少 CPU 或 RAM 或任何其他资源，而是更看重运行函数所需的时间，我们也只为这些函数的运行时间付费。 —— 《Serverless 架构应用开发指南》

而我们整个新架构的部署模式与 Serverless 是非常相近的，而我们系统还是一个完整的整体，可以不依赖于大量的线上服务。由于编译与开发一体，我们完成开发的同时，便完成了部署。从某种意义上来说，water 架构会比 Serverless 更快，还更适合于复杂架构应用。

PS：我将会用一篇新的文章来介绍这种模式。

### 模型-行为-分离

行为是围绕模型而构建的，在整个的机制之下，代码已经不再受模型的限制。接着，我们要解决的总是就是数据库的问题。

现今，我们的编程语言受限于模型，而模型受数据库影响。在这个情况下，如果我们不依赖于模型，那么我们需要设计一种新的机制，以让我们脱离数据库的束缚 —— 那大概只能发明一种新的数据库了。

回过头来看，数据库本身也是一种数据。那么，从现有的论文里去寻找一种更好的数据模式，也就变成了一件非常有意思的事。

## 其它

关于这个新的架构模式，我还在研究和思考中，未来依旧有很大的不确定性。但是，它真的非常好玩。

也欢迎到 GitHub 讨论和研究：[https://github.com/phodal/water](https://github.com/phodal/water)

相关文章：

 - 《云研发：研发即代码》：https://github.com/phodal/cloud-dev
 - 《万物代码化：从低代码、云开发到云研发》：https://www.phodal.com/blog/codify/
 - 《无代码编程》：https://www.phodal.com/blog/low-code-programming/
 - 《Charj —— 代码的代码化语言》：https://www.phodal.com/blog/charj-lang/

# 相关项目

## Darklang 语言

[https://darklang.com/](https://darklang.com/) is a new way of building serverless backends. Just code your backend, with no infra, framework or deployment nightmares. Build APIs, CRUD apps, internal tools and bots - whatever your backend needs.

GitHub： https://github.com/darklang/dark