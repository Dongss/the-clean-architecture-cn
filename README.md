# 整洁架构

[翻译自: The Clean Architecture by Robert C. Martin (Uncle Bob)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

![The Clean Architecture](https://blog.cleancoder.com/uncle-bob/images/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

在过去的几年中，我们看到有很多关系统架构的想法。这些包括：

* [Hexagonal Architecture（又称Ports and Adapters](http://alistair.cockburn.us/Hexagonal+architecture)，由Alistair Cockburn编写，Steve Freeman和Nat Pryce的著作[Growing Object Oriented Software](http://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627)中采纳了该观点
* Jeffrey Palermo的 [Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/)
* 我自己在去年的一篇博客中提出的 [Screaming Architecture](http://blog.cleancoders.com/2011-09-30-Screaming-Architecture) 
* James Coplien和Trygve Reenskaug 的 [DCI](http://www.amazon.com/Lean-Architecture-Agile-Software-Development/dp/0470684208/)
* Ivar Jacobson 的书 **Object Oriented Software Engineering: A Use-Case Driven Approach** 中的 [BCE](http://www.amazon.com/Object-Oriented-Software-Engineering-Approach/dp/0201544350)

尽管这些结构在细节上有所不同，但是它们非常相似。它们都有相同的目标，即关注点分离。他们都通过将软件划分为多个层来实现这种分离。每个层都有至少一层用于业务逻辑，另一层则用于接口。

基于这些架构的系统都有以下特点：

1. 不依赖框架。这些架构都不依赖于某些库或功能丰富的软件。这使您可以将此类框架当作工具，而不必使系统陷入这些框架有限的约束中。
2. 可测试的。可以在没有UI、数据库、Web服务器或任何其他外部元素的情况下测试业务逻辑。
3. 不依赖UI。可以在不修改系统其他部分的情况下轻松的修改UI。比如可以将一个Web UI替换成console UI，而不必修改业务逻辑。
4. 不依赖数据库。可以将Oracle或SQL Server换成Mongo，BigTable，CouchDB或其他东西。业务逻辑和数据库没有绑定。
5. 不依赖于外部系统。实际上，业务逻辑根本对外界一无所知。

## The Dependency Rule 依赖规则

同心圆代表软件的不同领域。通常，软件的级别越做越高。外圈是机制。内部圈子是政策。

使该体系结构起作用的最重要的规则是**The Dependency Rule**。按照该规则，源代码的依赖只能指向内部。内圈对于外圈的逻辑一无所知。特别是，在内圈中的代码不得提及在外圈中声明的名称。其中包括功能，类，变量或任何其他命名的软件实体。

同样，在外圈中使用的数据格式不应由内圈使用，特别是如果这些格式是由外圈中的框架生成的。我们不希望外圈中的任何东西影响内圈。

## Entities 实体

Entities封装企业级的业务逻辑。一个entity可以是带有方法的对象，也可以是一组数据结构和函数。Entities可以被企业中的不同应用程序使用。

如果不是企业级的，就只是一个独立的应用程序，那么entities就是该应用程序的业务逻辑对象。它们封装了最通用和最顶层的逻辑。当某些外部变化时，它们变化的可能性最小。例如，这些对象不会受页面导航或安全性更改的影响。任何对应用程序操作和交互的更改都不会影响entities层。

## Use Cases 用例

该层中的软件包含了**应用程序具体的**业务逻辑。它封装并实现了系统的所有use cases。这些use cases协调了entities之间的数据流，并指示这些entities使用其**企业级**的业务逻辑来实现use case的目标。

这一层的改动不应该影响到entities。这一层也不受外部改动的影响，比如数据库，UI，或者其他通用框架。该层与此类问题无关。

但是，我们确实希望对应用程序操作和交互的更改将影响use cases，并因此影响此层中的软件。如果use case的细节发生变化，那么此层中的某些代码肯定会受到影响。

## Interface Adapters 接口适配器

该层中的软件是一组适配器，可以将数据从对use cases和entities最友好的格式转换为对某些外部机构（如数据库或Web）最友好的格式。例如，正是这一层将完全包含GUI的MVC架构。Presenters，Views和Controllers都属于此处。Models可能只是从controllers传递到use cases，然后再从use cases传递到presenters和views的数据结构。

同样，在这一层中，数据从entities和use cases的友好形式转换为最适合使用任何持久性框架的形式，比如数据库。这个圈子内的任何代码都不应该对数据库一无所知。如果数据库是SQL数据库，则所有SQL都应限于该层，尤其是该层中与数据库有关的部分。

在该层中，还需要其他任何适配器，以将数据从某种外部形式（例如外部服务）转换为use cases和entities使用的内部形式。

## Frameworks and Drivers 框架和驱动程序

最外层通常由框架和工具组成，例如数据库，Web框架等。通常，除了与内圈通讯的连接代码以外，在该层中不需要编写太多代码。

这层是所有细节所在的地方，Web是一个细节，数据库是一个细节。我们将这些东西放在外面，使他们不会对系统造成太大伤害。

## Only Four Circles? 只有四个圈？

不，这些圈只是原理示意图。你可能会发现实际需要比这更多的圈。没有规则说必须始终只有这四个。然而**The Dependency Rule**始终适用，代码的依赖始终指向内部。越往内圈抽象程度越高。最外层是底层的具体细节。从外圈到内圈，该软件变得越来越抽象，并封装了更高级别的策略。最内圈是最通用的。

## Crossing boundaries 跨越边界

图的右下方是我们如何越过圆边界的示例，它显示了Controllers和Presenters与下一层的Use Cases进行通信。注意控制的流程：它从controller开始，经过use case，然后在presenter执行结束。也要主要代码依赖，他们每个都向内指向use cases。

我们通常通过使用[Dependency Inversion Principle](http://en.wikipedia.org/wiki/Dependency_inversion_principle)(依赖倒置原则)来解决这种明显的矛盾。比如在Java语言中，我们会安排接口和继承关系，以使源代码依赖项在跨边界时正确的做出反向控制流。

比如，假设use case需要调用presenter，然而这种调用不能直接进行，因为这回违反**The Dependency Rule**：外圈的任何名称不能在内圈中提及。因此，我们在内圈中的一个use case调用一个接口（在此处作为为Use Case Output Port），并在外圈中有一个presenter来实现它。

在体系中要使用相同的技术跨边界。我们利用动态多态来创建与控制流相反的代码依赖项，以便无论控制流向哪个方向运行，我们都可以遵循**The Dependency Rule**。

## What data crosses the boundaries 哪些数据跨越了边界

通常，跨越边界的数据是简单的数据结构，可以使用基本结构或简单的数据传输对象，或者数据只是函数调用的参数，或者可以把数据使用hashmap结构，或者把数据构建成一个对象。重要的是，跨边界传递的数据是独立的、简单的数据结构，不要偷懒去传递**Entities**或者数据库行数据。不要让数据结构有任何违反**The Dependency Rule**的依赖性。

比如，很多数据库框架都会对查询语句返回一个比较友好的数据结构，暂且称为RowStructure，RowStructure是不应该跨边界传输的，那样会违反**The Dependency Rule**, 因为这要求内圈必须要了解一些外圈的东西。

所以当我们跨边界传输数据的时候，通常是使用对内圈友好的数据形式。

## Conclusion 结论

遵循这些简单规则并不难，并且可以避免很多麻烦。将软件分成几层，遵循**The Dependency Rule**，就可以创建本质上可测试的系统，并且包含很多潜在的好处。当系统的任何外部部分（例如数据库或Web框架）变得过时时，都可以用最少的麻烦替换那些过时的元素。