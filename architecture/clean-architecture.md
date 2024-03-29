今天这篇文章主要是翻译`Robert C. Martin`提出来的整洁架构文章，该文章发表于2012-8-13，该架构指导我们如何把所有的概念、规则和模式整合起来，形成一种标准实现套路。如果英文阅读能力好的同学可以直接阅读英文版[The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)。

以下为正文：

![clear-architecture](../public/clear-architecture.png)

在过去的几年中，我们看到了一系列关于系统架构的想法，包括:
- [六边形架构](http://alistair.cockburn.us/Hexagonal+architecture)(也称端口和适配器)，Steve Freeman和Nat Pryce在他们的精彩著作《成长的面向对象软件》中采用了该体系结构
- Jeffrey Palermo的洋葱架构,可以看[这篇](./%E6%9E%B6%E6%9E%84%E6%BC%94%E5%8F%98%E5%8F%B2.md#洋葱架构-2008)文章介绍
- 我的博客去年提出的[Screaming Architecture](http://blog.cleancoders.com/2011-09-30-Screaming-Architecture)
- James Coplien, Trygve Reenskaug的[DCI](http://www.amazon.com/Lean-Architecture-Agile-Software-Development/dp/0470684208/)
- Ivar Jacobson在其著作《面向对象软件工程：用例驱动方法》中的[BCE](http://www.amazon.com/Object-Oriented-Software-Engineering-Approach/dp/0201544350)

尽管这些架构在细节上有所不同，但它们非常相似。它们都有相同的目标，即关注点分离。它们都通过将软件划分为层次来实现这种分离。每种架构至少有一个层次用于业务规则，另一个用于接口。

这些架构各自产生的系统特点是：
1. 独立于框架：该架构不依赖于某些功能丰富的软件库的存在。这使得你可以将这样的框架作为工具使用，而不是让你的系统受限于它们的局限性。
2. 可测试：业务规则可以在不依赖UI、数据库、Web服务器或任何其他外部元素的情况下进行测试。
3. 独立于UI：UI可以轻松更改，而不需要更改系统的其余部分。例如，Web UI可以替换为控制台UI，而不更改业务规则。
4. 独立于数据库：你可以更换Oracle或SQL Server，使用Mongo、BigTable、CouchDB或其他数据库。你的业务规则不绑定在数据库上。
5. 独立于任何外部机构：实际上，你的业务规则对外部世界一无所知。

本文顶部的图表试图将这些架构整合成一个单一的可执行理念。

### 依赖性规则 
同心圆代表软件的不同领域。一般来说，越往里走，软件的级别越高。外圈是机制，内圈是策略。

使这种架构起作用的主要规则是**依赖性规则**。这个规则指出，**源代码依赖**只能指向**内部**。内圈中的任何东西都不能知道外圈中的任何事情。特别是，外圈中声明的东西的名字绝不能被内圈中的代码提及。这包括函数、类、变量或任何其他命名的软件实体。

同样地，外圈中使用的数据格式也不应该被内圈使用，尤其是如果这些格式是由外圈中的框架生成的。我们不想让外圈中的任何东西影响到内圈。


### 实体层
实体封装了**整个系统**内的业务规则。一个实体可以是一个带有方法的对象，也可以是一组数据结构和方法。只要实体可以被企业中的许多不同应用程序使用，这并不重要。

如果你没有整个系统，只是编写一个单一的应用程序，那么这些实体就是应用程序的业务对象。它们封装了最通用和最高级别的规则。当外部发生变化时，这些对象的改变可能性最小。例如，你不会期望这些对象受到页面导航或安全性变化的影响。任何特定应用程序的操作变化都不应该影响实体层。
### 用例层
这一层通常包含的是特定应用场景下的业务逻辑。这里面封装并实现了整个系统的所有用例。该层控制所有流向和流出实体层的数据流，并使用核心的实体及其业务规则来完成业务需求。

我们不期望这一层的变化影响实体。同时，我们也不期望这一层受到数据库、UI或任何常见框架等外部因素变化的影响。这一层与这些关注点是隔离的。

然而，我们预计应用操作的变化将会影响到用例，因此也会影响到这一层的软件。如果用例的细节发生变化，那么这一层的某些代码肯定会受到影响。

### 接口适配器
在整个软件中它们是一组适配器，它们负责把数据从用例和实体最方便的格式，转换成数据库或Web端等外部机构最方便的格式。比如，这层就会完全包含GUI的MVC架构。展示器、视图和控制器都放在这里。模型可能只是数据结构，从控制器传给用例，然后从用例传回展示器和视图。

同样，在这一层中，数据从最适合实体和用例的形式转换为最适合所使用的任何持久化框架的形式，即数据库。这个圆圈内部的所有代码都不应该知道有关数据库的任何事情。如果数据库是SQL数据库，那么所有的SQL应该限制在这一层，特别是与数据库相关的这一层的部分。

在这一层中还包括任何其他必要的适配器，用于将数据从某些外部形式（如外部服务）转换为用例和实体使用的内部形式。

### 框架和驱动程序
 最外层通常由框架和工具组成，如数据库、Web框架等。通常你不会在这一层编写太多代码，除了与向内下一个圆圈通信的粘合代码。

这一层是所有细节所在的地方。Web是一个细节。数据库是一个细节。我们把这些东西放在外面，这样它们就不会造成太大的伤害。大多是一些用来跟内层通信的胶水代码。

### 只有四层吗？
不是的，这些圈只是示意图。你可能会发现你需要的不止这四个。没有规定说你必须总是只有这四个。然而，**依赖性规则**始终适用。源代码依赖总是指向内部。当你向内移动时，抽象层次会增加。最外层的一般是比较低级别的抽象封装。当你向内移动时，软件变得更加抽象，并封装了更高级别的策略。最里面的圆是最通用的。

### 跨越边界
 在图表的右下角是一个如何跨越圆圈边界的例子。它展示了控制器和展示器如何与下一层的用例进行通信。注意控制流。它从控制器开始，通过用例，然后在展示器中结束。还要注意源代码依赖。它们每一个都指向内部的用例。

我们通常通过使用[依赖倒置原则](http://en.wikipedia.org/wiki/Dependency_inversion_principle)来解决这个明显的矛盾。例如，在Java这样的语言中，我们会安排接口和继承关系，使得源代码依赖在跨越边界时正好与控制流相反。

例如，考虑用例需要调用展示器。然而，这个调用不能直接进行，因为这会违反依赖性规则：内部圈不能提及外部圈的名字。所以我们让用例调用内部圈的一个接口（在这里显示为用例输出端口），让外部圈的展示器实现它。

同样的技术在架构中用于跨越所有边界。我们利用动态多态性创建源代码依赖，以反对控制流，这样无论控制流的方向如何，我们都能遵守依赖性规则。

### 什么是数据跨越边界 
通常跨越边界的数据是简单的数据结构。如果你愿意，可以使用基本的结构体或简单的数据传输对象。或者数据可以是函数调用的参数。或者你可以把它打包到哈希映射中，或者构建成对象。重要的是，孤立、简单的数据结构被传递过边界。我们不想作弊，传递实体或数据库行。我们不想让数据结构有任何违反依赖性规则的依赖。

例如，许多数据库框架在响应查询时返回一个方便的数据格式。我们可能称之为行结构。我们不想把那个行结构向内跨越边界传递。这将违反依赖性规则，因为它将迫使内部圈了解外部圈的一些信息。

所以，当我们跨边界传递数据时，它总是以最方便内部圈的格式传递。

### 结论 
遵守这些简单的规则并不难，而且可以为您节省很多未知的麻烦。通过将软件分层，并遵守依赖性规则，您将创建一个本质上可测试的系统，这暗示了所有的好处。当系统的任何外部部分变得过时，比如数据库，或者Web框架，您可以最小化麻烦地替换这些过时的元素。











