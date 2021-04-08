---
title: "领域驱动设计中的分层架构探讨"
linkTitle: "DDD中的分层架构探讨"
date: 2016-08-25
description: >
  本文从常见的分层架构和基础概念入手，对设计原则、各层职责、应包含的内容、依赖关系、优缺点等进行分析探讨
---

> 在分解复杂的软件系统时，分层是我们最常用的手段之一。然而，一千个读者心中有一千个哈姆雷特，对于建立哪些层次以及各层的具体职责，大家也往往持有不同的见解。
在领域驱动设计中，层次和包的划分看起来与我们的结构又有一定区别，加上对领域中一些概念的理解不一致，使得局面更加混乱。

本文从常见的分层架构和基础概念入手，对设计原则、各层职责、应包含的内容、依赖关系、优缺点等进行分析探讨，尝试解决如下的一些问题。

+ 领域驱动设计中是如何进行分层的？和传统的分层模式有什么区别和联系？
+ 领域层的一些核心内容？领域模型到底是什么？
+ 各层如何抉择和裁剪？必要性（是否可选）、优缺点？使用场景是什么？
+ 各层一些常见的模式（如`MVC`、`CQRS`、数据映射器、`Repository`等)
+ 常见问题（领域层和服务层区别？外部服务调用在哪一层？贫血失血模型比较？等）

## 为什么要分层 - Why?

> All problems in computer science can be solved by another level of indirection, except of course for the problem of too many indirections. - David Wheeler

软件设计中分层的设计随处可见，但是分层能带来什么好处呢？或者说，我们为什么要考虑分层架构呢？

由于现实世界的复杂性，分层可以提供一个相对高层的视角来 **分解和简化我们的问题**，此外分层也可带来 **可测试性**、 **可维护性**、 **灵活性**、 **可扩展性**等方面的好处。

+ 简化复杂性，关注点分离（设计中每个部分都得到单独的关注，修改的影响也限定在某个范围内），结构清晰；
+ 降低耦合度，隔离层次，降低依赖（上层无需关注下层具体实现），利于分工、测试和维护（可维护性）；
+ 提高灵活性，可以灵活替换某层的实现；
+ 提高扩展性，方便实现分布式部署；

看起来十分简单，好像就是把系统划分为一定的层数，并把他们堆叠组织起来。但是，当落实到具体的实践时， **如何划分、各层存在的意义、如何取舍以及相应的依赖关系** 却并没有想象中那么容易， **边界的重合部分、不同场景下关注点、层次内部的具体分解以及层次的粒度** 等都是我们需要考虑的问题。

## 什么是分层架构 - What?

### 分层的历史

最广为人知的应该就是经典的三层架构([Three-Tier Architecture](https://en.wikipedia.org/wiki/Multitier_architecture))：展示层(`presentation/user interface`)、业务逻辑层(`business logic`)、数据访问层(`data access`)。

`Martin Fowler`在《企业应用架构模式》(`Patterns of Enterprise Application Architecture`)中也是以类似的三层(`Three Principal Layers`)进行展开的: 表现层(`Presentation`)，领域层(`Domain`)，数据源层(`Data Source`)。

**注**: 由于`Tier`只是表示是物理上的分离（最初提出并应用于`C/S`架构时是部署在不同的机器上的），`Layer`才是表示逻辑上的架构，所以后续我们也将使用`Layer`一词。

然而，其中又存在各种各样的变体，这也给我们的理解造成一定的混乱（如下表所示）。

|Martin Fowler|Martin Fowler(Detail)              |Brown               |Core J2EE    |Microsoft DNA|Marinescu    |Nilsson           |
|------------|------------------------------------|--------------------|-------------|-------------|-------------|------------------|
|            |Presentation that runs on client    |                    |Client       |             |             |                  |
|Presentation|Presentation                        |Presentation        |Presentation |Presentation |Presentation |Consumer          |
|            |Presentation(Application Controller)|Controller/mediator |             |             |Application  |Consumer helper   |
|Domain      |Domain(Service Layer)               |Domain              |Business     |Business     |Services     |Application       |
|            |Domain(Domain Model)                |                    |             |             |Domain       |Domain            |
|            |Data source(Data Mapper)            |Data mapping        |             |Data access  |             |                  |
|Data source |Data source                         |Data source         |Integration  |             |Persistence  |Persistence access|
|            |External resource                   |                    |Resource     |             |             |                  |

面对如此多的分层架构，我们不禁思考，他们 **分层的依据又是什么呢？能否抽象出一些相同点和不同点？又该在什么时候加入哪些合适的中间层？在实践中我们又该采取怎样的架构呢？** 

### 分层的本质

我们不妨先 **回到问题的起点，思考到底什么是分层**。
分层其实是 **把一系列相同或相似的对象进行分类并放在同一层**，然后 **根据他们之间的依赖关系再确定上下层次关系**。
可以看出，分层的核心在于 **分类** 和 **关联**(事物的两面性： **区别和联系**)。

+ 分类：确定 **划分标准**，将相关对象进行 **抽象（分离变与不变，提取共同点）并归类**，对层之间进行 **隔离**，并保持层内元素是高度 **内聚**；
+ 关联：层与层以及外部是存在联系的，需要关注不同层间的 **依赖与通信**，保持层间的 **松散耦合**；

通常，我们可以将系统划分为变化较大的 **业务** 部分和相对稳定的 **技术** 部分；对于业务来说，又可划分为展示的部分（前台）和内部处理逻辑（后台）两大部分；展示又可分为数据/页面部分和接口部分。如此不断的进行细分和抽象，我们可以迭代出更细粒度的分类/层次，如下列表所示：

+ 业务：需要重点关注，我们的目的也是分离出具体的业务领域逻辑
  + 外部展示( **表现层/接口层**, `Presentation Layer`/`User Interface Layer`/`View Layer`)
      - 数据、页面(`web`)
      - 远程接口(`interface/api`)
  + 内部逻辑处理
      - 应用逻辑( **应用层/服务层**,`Application Layer`/`Service Layer`/`Controller Layer`)
      - 具体业务逻辑( **领域层**, `Business Layer`/`Business Logic Layer`/`Domain Layer`)
+ 技术：相对稳定，具体业务无关( **基础设施层**)
  + 数据访问（ **数据访问层**, `Data Access Layer`）
  + 日志、安全、异常、缓存等

当然，分类并不唯一， **基于不同的视角我们可能会有不同的分类标准**。比如数据访问层也可以归类到业务相关/内部逻辑处理的部分，因为可能涉及到一些对具体业务表的操作（上面的分类标准里的业务指的是具体的业务逻辑，而对数据库的增删改查是不涉及具体的业务规则的，所以归类到了基础设施层）。

此外，根据 **问题领域和解决方案的复杂程度**，我们可以有不同的层次。比如在业务不太复杂时，我们可以把应用层和领域层合并为一层。

**注**: 我们的分类更多的是针对于一些具体的业务系统，如企业应用。对于一些其他类型的应用（如操作系统、网络协议、中间件等），需要根据具体情况进行划分。

## 如何分层 - How?

上面我们在分析分层的本质时也提到了一些基本的层次和分类标准，但那只是一个非常粗粒度的划分（可以说是单纯的分类）。
在实际 **决策** 时，我们需要知道 **各层的职责、意义以及相应的场景**；而落实到 **代码** 层面时，我们还需知道 **各层所包含的具体内容、各层的一些常见的具体策略/模式、层次之间的交互/依赖关系**。

### 基本目标与原则

在开始之前，我们先来考虑下分层的目标和应遵循的基本原则。

前面我们也提到了分层的一些好处（如简化复杂性、降低耦合度），在具体实施之前，我们还需要进一步思考分层为什么分层能带来这些好处。比如复杂性体现在哪？分层是如何降低这种复杂性的？系统又是如何解耦的？

我们总是希望设计出 **高内聚、低耦合、易维护、可扩展** 软件，分层就是达到高内聚低耦合的手段之一。

+ 高内聚(同一层): 单一职责
+ 低耦合(层与层之间): 每层只能依赖于同层以及其下层，每层以接口方式供上层调用；

原则其实可以看做一种 **高层指导规范**，可以帮助我们避免一些常见的坑或者帮助我们更好的实施以得到更好的效果。

+ 关注点分离（`Separation of Concerns`）: 使设计中每个部分都得到单独的关注，在分离的同事，也需要维持系统内部复杂的交互关系

+ 最少知识原则(`Least Knowledge Principle`/`Law of Demeter`)

此外，还有一些通用的面向对象设计原则，如[S.O.L.I.D](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)原则。

+ 单一职责(`Single Responsibility Principle`): 每个类应该只有一个独一无二的职责，这条针对类的原则同样适用于每一层；
+ 开-闭原则(`Open Close Principle`): 对象或实体应该对扩展开放，对修改封闭；
+ 里氏替换原则(`Liskov Substitution Principle`): 应用程序应该依赖于抽象（基类或者接口），而不是具体实现
+ 接口分离原则(`Interface Segregation Principle`): 接口的职责也应该是单一的，接口中应该包含哪些方法，需要进行严格的评估
+ 依赖倒置(`Dependency Inversion Principle`): 抽象不能依赖于具体，而具体则应该依赖于抽象。类之间的直接依赖应该用抽象来取代

接下来，我们再来深入分析下分层在领域设计中的一些具体实践。

### DDD经典分层架构

首先我们来看一下Evans在《领域驱动设计》(`Domain-Driven Design`)中提到的分层架构。

![领域驱动设计分层架构](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/3a1202c8e7ac1972f7692c8aaedcf40a)

<font color="red">**问: 为什么要分成这样的四层？**</font><br/>
这个问题其实有很多种解释，可以从分层的目的来展开，也可以从三层架构的演进来阐述，还可以从问题域本身来回答。
我们在前面也有提到过，分层主要目的是为了简化复杂性，系统中最复杂的部分应该就是我们的业务逻辑。当系统交互或者工作流比较复杂时，我们会考虑从业务逻辑中抽出这部分作为应用层。
除了数据访问，我们可能还需要一些消息通信、安全处理等基础通用技术，我们把这些统称为基础设施。

<font color="red">**问: 对于层次间依赖，是否允许第N层直接访问第N-2、N-3…层？**</font>
分层架构有两种典型的风格：

+ **严格分层**(`Strict Layers Architecture`): 只能访问同层和直接下层； **降低了层之间的耦合性**，对低层的修改不会对整个系统造成广泛的影响，但可能造成 **无意义的代码**；
+ **灵活分层**(`Relaxed Layers Architecture`): 允许访问同层和所有下层；无需引入过多的请求，性能相对较高，缺点是可能造成 **混乱**；

虽然`DDD`推荐灵活的分层模式，但是这种做法在实际工程中这个比较容易造成混乱和难以管理。所以一般实践可能会 **首先使用严格的分层，当某层的大多数代码基本没有任何业务逻辑（即污水池反模式）时，才考虑重构并开放该层。**

<font color="red">**问: 怎么处理分层和分割的关系？**</font>
其实分层是一种 **横向** 的切割，而分割（划分模块）是一种基于功能上的 **纵向** 切割。
我们往往需要结合这两种方式对系统进行分解。
在划分`package`时，我们常常能看到两种做法，一种是每层作为一个包名（如`domain`，`dao`等），里面放置各个对应的功能。另外一种就是以功能来作为包名，里面包含各种实现。
一般推荐 **先按功能进行划分模块，然后模块里面再进行进行分层**。功能相对内聚，层次也比较清晰，同时也利于后续的拆分或者微服务化。
在纵向分割时，同样存在一些不同的视角（例如物流业务可以按海运、陆运、空运、快递等业务来划分模块，也可以按方案查询、下单、跟单、结算等来划分模块），需要架构师根据变化性以及共同点等去进行权衡。

#### <font color="blue">用户界面层/表示层(User Interface/Presentation Layer)</font>

> Responsible for **showing information to the user** and **interpreting the user's commands**. The external actor might sometimes be another computer system rather than a human user. - Eric Evans

**用户界面层**负责 **向用户显示信息和解释用户指令**。这里指的用户 **可以是另一个计算机系统**，不一定是使用用户界面的人。

> This layer holds everything that **interacts with other systems**, such as web services, RMI interfaces or web applications, and batch processing frontends. It **handles interpretation, validation and translation of incoming data**. It also **handles serialization of outgoing data**, such as HTML or XML across HTTP to web browsers or web service clients, or DTO classes and distributed facade interfaces for remote Java clients. - DDDSample

该层包含与其他应用系统（如Web服务，RMI接口或Web应用程序以及批处理前端）**交互的接口与通信设施**。
它负责 **输入参数的解释、验证以及转换**。另外，它也负责 **输出参数的序列化**，如通过HTTP协议向Web浏览器或Web服务客户端传输HTML或XML，或远程Java客户端的DTO类和远程外观接口的序列化。

可以看出，该层的主要职责是与外部用户（包括web服务、其他系统）交互，如接受用户的反馈，展示必要的数据信息。
主要包含`web`部分和远程服务部分等。
`web`部分一般包含常见的`Servlet`, `Controller`等组件，而远程接口部分主要由`Façade`、`DTO`和`Assembler`等构成(J2EE设计模式之一，也是`PoEEA`中提到的分布模式)。

+ `Façade`远程外观，一个粗粒度的外观，不含任何领域逻辑
+ `DTO` 数据传输对象(`Data Transfer Object`)
+ `Assembler` 对象组装器，负责数据传输对象与领域对象相互转换，不对外暴露

![RemotePattern](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/7eb48b99c15190ac036fbf9feaa5ca41)

<font color="red">**问: 用户界面层指的就是显示相关的东西吗？**</font>
这里的用户其实不一定指人，还包括计算机系统，所以这里的`UI`不单是显示，而可以理解为更广义的接口（从外部来看，可以看到的东西）。
对于一个模块来说，从外部能看到的就是其所暴露的接口；对于`web`页面，我们也可以理解为相应的`web`接口。
**注意**：这`web`部分和远程接口部分可能分别在提供访问页面的`web`服务器上和专门提供服务（如`HSF`）的服务器上（逻辑上还是属于同一层的）。
此外，现实中我们一般会把`Façade`服务的接口和`DTO`独立出来，以二方库的形式供外部使用。

<font color="red">**问: 用户界面层和`MVC`是什么关系？和其中的`View`是通一个概念吗？**</font>
`MVC`只是展现层的一种实现模式(类似的有`MVP`或者`MVVM`等)，是为了实现对该层的一个细分（对数据与行为进行解耦）。
`View`只是其中的一小部分，`Servlet`、`Controller`等都是属于用户界面层。

<font color="red">**问: 参数的校验为什么在用户界面层？领域层的校验和这儿的校验有什么不同吗？**</font>
校验应该取决于校验的内容，一般推荐尽早校验，不过这里主要是进行一些简单的、不涉及业务规则的校验。具体的业务规则的校验放在领域层。

<font color="red">**问: 为什么需要`Façade`？这里的远程外观和普通的外观有什么区别吗？**</font>
由于远程通信的开销，我们希望减少系统交互次数，远程外观定义了一个对外的高层接口（粗粒度），屏蔽了相应内部细节，简化了外部调用。
在复杂的情况中，单个的远程外观可以充当许多细粒度对象的一个远程入口，减少调用次数、简化交互难度。

<font color="red">**问: 为什么需要`DTO`？`DTO`和`VO`是同一个东西吗？**</font>
领域对象关系比较复杂，很难序列化，而且用户很多时候并不需要整个模型，大部分时候需要的只是其中的一部分内容，`DTO`可以有效减少网络调用的开销。此外，领域模型内部的逻辑也无需暴露给外部。
`DTO`一般用于远程服务，如果是内部使用的话，一般可以直接使用领域对象。
`DTO`和`VO`是有区别的，有些团队在这里使用`VO`(值对象)，但不推荐，因为`VO`在领域模型中有着特殊的含义。

<font color="red">**问: 为什么需要`Assembler`？**</font>
主要目的是解耦，负责数据传输对象和领域对象之间的相互转换。`BeanUtils`也可以做到相应的功能（`dozer`相对好一些），不过`Assembler`更为清晰，安全与可控，缺点在于手工代码量稍多。

#### <font color="blue">应用层(Application Layer)</font>

> **Defines the jobs the software is supposed to do** and directs the expressive domain objects to work out problems. The tasks this layer is responsible for are meaningful to the business or **necessary for interaction with the application layers of other systems**.
This layer is kept **thin**. It does not contain business rules or knowledge, but **only coordinates tasks and delegates work to collaborations of domain objects in the next layer down**. It does not have state reflecting the business situation, but it can have state that reflects the progress of a task for the user or the program. - Eric Evans

应用层定义了 **软件要完成的任务**，并且指挥表达领域概念的对象来解决问题。该层所负责的工作对业务来说意义重大，也是 **与其他系统的应用层进行交互的必要通道**。
应用层要尽量简单。它 **不包含任何业务规则或知识**， **只是为下一层的领域对象协助任务、委托工作**。它没有反映业务情况的状态，但它可以具有反映用户或程序的某个任务的进展状态。

> The application layer is responsible for **driving the workflow of the application**, **matching the use cases** at hand. These operatios are interface-independent and can be both synchronous or message-driven. This layer is well suited for **spanning transactions, high-level logging and security**.
> The application layer is **thin** in terms of domain logic - it merely **coordinates the domain layer objects to perform the actual work**. - DDDSample

应用层主要负责 **组织整个应用的流程**， **是面向用例设计的**。该层非常适合 **处理事务，日志和安全** 等。相对于领域层，应用层应该是 **很薄的一层**。它只是 **协调领域层对象执行实际的工作**。

综上所述，应用层是 **表达`user case`和`user story`的主要手段**，主要用于 **协调领域模型与其它应用组件的工作**(并不处理业务逻辑)。
应用层中主要组件是`Service`，因为主要职责是协调各组件工作，所以通常会与多个组件交互，如其他`Service`，领域对象，`Repostitory`等。
一种比较常见的做法是：应用层通常接受来自用户界面层的参数，再通过`Repostitory`获取到聚合示例，然后执行相应的命令操作(很薄的一层)。

<font color="red">**问: 为什么要有应用层？**</font>
业务比较复杂时，我们会从业务逻辑中拆分出应用层和领域层。
如果在领域对象中事项针对具体应用的逻辑，会降低在应用之间的可重用性。
此外，如果将来需要加工作流之类的工具来实现应用逻辑，如果之前是混杂在一起的话则不好拆分。

<font color="red">**问: Application层的Service和日常TransactionScript风格的Service是一个东西吗？**</font>
`TransactionScript`风格的`Service`主要用来实现业务逻辑，所以比较重。而在领域驱动设计中`Application`层的`Service`只负责协调并委派业务逻辑给领域对象进行处理，是很薄的一层。

<font color="red">**问: 这里的应用层和 PoEAA 中的服务层(Service Layer)是同一个东西吗？**</font>
> Defines an **application's boundary** with a layer of services that establishes a set of available operations and coordinates the application's response in each operation. - PoEAA

`PoEEA`中所说的服务层定义了 **应用的边界和从接口客户层角度所能看到的可用操作集**。它 **封装了应用的业务逻辑、事务控制及其操作实现中的相应协调**。
可以看出，两者基本是类似的概念。

#### <font color="blue">领域层/模型层(Domain/Model Layer)</font>

> Responsible for **representing concepts of the business, information about the business situation, and business rules**. State that reflects the business situation is controlled and used here, even though the technical details of storing it are delegated to the infrastructure. This layer is the heart of business software. - Eric Evans

领域层主要负责 **表达业务概念，业务状态信息和业务规则**。

> The domain layer is the **heart of the software**, and this is where the interesting stuff happens. There is **one package per aggregate**, and to each aggregate belongs **entities, value objects, domain events, a repository interface and sometimes factories**. - DDDSample


`Domain`层是整个系统的 **核心** 层， **几乎全部的业务逻辑会在该层实现**。

领域模型层（`Domain Model Layer`）主要包含以下的内容：

+ 实体(Entities):具有唯一标识的对象
+ 值对象(Value Objects): 无需唯一标识
+ 领域服务(Domain Services): 一些行为无法归类到实体对象或值对象上，本质是一些操作，而非事物
+ 聚合/聚合根(Aggregates & Aggregate Roots): 聚合是指一组具有内聚关系的相关对象的集合，每个聚合都有一个`root`和`boundary`
+ 工厂(Factories): 创建复杂对象，隐藏创建细节
+ 仓储(Repository): 提供查找和持久化对象的方法

关于各个元素的具体含义、职责以及相关误区，可参考[领域建模核心概念解析](http://www.atatech.org/articles/64749).
对于这些具体的对象，可定义一些标准领的`Annotation`来规范。

<font color="red">**问: 领域层怎么又这么多好多平常多见不到的对象？**</font>
组织领域逻辑一般有三种组织方式：事务脚本(`Transaction Script`)、领域模型(`Domain Model`)、表模块(`Table Module`)。
这些对象都是采用领域模型的方法时的工具，其他方式并不需要。

<font color="red">**问: 为什么需要需要 Aggregate, Factory, Repostory？**</font>
在复杂的关联关系中，对象的管理和保证数据的一致性比较困难，因此我们使用`Aggregate`来管理一组相关的对象，降低对象使用的复杂性。
但这样也造成的对象创建和持久化对象可能比较复杂，所以我们引入`Factory`创建复杂对象和`Aggregate`，封装内部复杂性；并使用`Repostory`来提供查找和持久化对象。
`Factory`和`Repostory`本身并不是来源于领域，但他们在领域设计中扮演了重要的角色（辅助管理具体的领域对象），因此我们也将他们列入了领域模型。

<font color="red">**问: 怎么区分一个服务到底是应用服务(`Application Services`)还是领域服务(`Domain Services`)？**</font>
这点其实前面介绍`Application`层有提到一些，主要还是看是否是和业务相关。

<font color="red">**问: 领域服务(`Domain Services`)和我们日常`TransactionScript`风格的`Service`是类似的概念吗？**</font>
表面看来，两者确实很像。但`Domain Services`指的是领域中的一些概念不太适合建模为对象（无法归类为实体对象或值对象），因为他们本质上是一些操作/动作，而不是事物。

<font color="red">**问: 领域模型就是把行为放到模型中吗？贫血模型和充血模型的本质区别？**</font>
看起来领域模型和我们目前做的区别就是在对象里面添加了行为。可实际真的是这样吗？
方法在模型里面和外面真有这么大的差别吗？如果把行为都放到模型中，那么模型会不会变得很臃肿？

<font color="red">**问: `Repository`为什么只是一个接口放在领域层？**</font>
`Repository`接口存在于`Domain`层，而具体实现存在于`Infrastructure`层，这样的设计乍看起来有点奇怪。
不过我们可以从`Repository`的职责和领域层的目的两方面去思考这个问题。
`Repository`主要负责数据库的存储和读取，但如何查找和持久化属于具体的技术实现，而领域层的目的之一是让我们更加关注业务的具体业务逻辑，因此对于`Repository`具体实现放在了`Infrastructure`层。
另一方面，在具有复杂关系模型中，我们引入了`Aggregate`来降低对象使用的复杂性，那如何管理这些这些`Aggregate`是我们需要关注的R，所以我们在领域模型中需要知道`Repository`接口（不是单纯的与数据库的映射，一般只有只有`Aggregate Root`才有对应的`Repository`）。

#### <font color="blue">基础设施层(Infrastructure Layer)</font>

> **Provides generic technical capabilities** that support the higher layers: message sending for the application, persistence for the domain, drawing widgets for the UI, and so on. - Eric Evans

基础设施层为上面各层提供通用的技术能力：为应用层传递消息，为领域层提供持久化机制，为用户界面层绘制屏幕组件。

> The infrastructure consists of everything that exists independently of our application: external libraries, database engine, application server, messaging backend and so on.
> Also, we consider code and configuration files that glues the other layers to the infrastructure as part of the infrastructure layer. Looking for example at the persistence aspect, the database schema definition, Hibernate configuration and mapping files and implementations of the repository interfaces are part of the infrastructure layer. - DDDSample

基础设施层以不同的方式支持所有三个层，促进层之间的通信。
基础设施包括独立于我们的应用程序存在的一切：外部库，数据库引擎，应用程序服务器，消息后端等。

作为基础设施层，`Infrastructure`为`Interfaces`、`Application`和`Domain`三层提供支撑。所有与具体平台、框架相关的实现会在`Infrastructure`中提供，避免三层特别是`Domain`层掺杂进这些实现，从而“污染”领域模型。`Infrastructure`中最常见的一类设施是对象持久化的具体实现。

<font color="red">**问: `Repository`作用是什么？和`DAO`的关系**</font>
之前对`Repository`也曾有过误解（在我们的系统中有一个`repository`层位于`dao`和`service`之间）。
`DAO`主要是从数据库表的角度来看待问题的，并且提供`CRUD`操作（只是对数据库表的一个封装），是一种面向数据处理的风格（事务脚本）；
而`Repository`(资源库)和`Data Mapper`(数据映射器)更加面向对象，通常用于领域模型中。
因为数据访问层的暴露可能会破坏对象的封装性，对象的关系和数据一致性也难以维护，所以 **应该尽量避免在领域模型中使用`DAO`模式，推荐使用聚合本身来管理业务逻辑。**

<font color="red">**问: 为什么没看到`PO`相关的对象？不需要将领域对象转成数据库对象吗？**</font>
`PO`即`Persistent Object`数据存储对象（公司内部很多实用数据对象`DO/Data Object`来标识），只是一种单纯的数据容器。

#### <font color="blue">小结</font>

`DDD`架构(细化)

![DDD_Sample_Layered_Architecture](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/76497cf6ff65c0b1b8e98ffbd74f08ca)

`ddd-sample`代码结构(抽象)

```
└── dddsample 项目名(一般我们系统采用多模块，这里可能是其中一个子模块)
    ├── interfaces 用户界面层（我们实践中一般会拆分到web工程和二方库中）
    │   └── [module] 子模块（多个更小的子域）
    │       ├── facade
    │       │   ├── XxxServiceFacade.java 远程外观接口（接口和相关DTO一般会以二方库形式提供）
    │       │   ├── dto
    │       │   │   └── XxxDTO.java 数据传输对象
    │       │   └── internal 内部实现 （服务的具体实现）
    │       │       ├── XxxServiceFacadeImpl.java 远程外观实现
    │       │       └── assembler
    │       │           └── XxxDTOAssembler.java 数据传输对象装配器
    │       └── web （实际中一般拆出独立的web项目）
    │           ├── XxxServlet.java
    │           ├── XxxCommand.java
    │           └── XxxController.java
    ├── application 应用层
    │   ├── XxxService.java
    │   ├── impl
    │   │   └── XxxServiceImpl.java
    │   └── util
    │       └── XxxUtil.java
    ├── domain 领域层
    │   ├── model
    │   │   └── [module] 子模块（多个更小的子域）
    │   │       ├── Xxx.java 领域对象
    │   │       └── XxxRepository.java 聚合根仓储对象
    │   └── service
    │       └── XxxService.java 并不是和领域对象对应的Service
    └── infrastructure 基础设施层
        ├── messaging
        │   └── jms
        │       └── XxxConsumer.java
        ├── persistence
        │   └── hibernate
        │       └── XxxRepositoryHibernate.java 仓储的具体实现类
        └── [module] 子模块（多个更小的子域）
            └── ExternalXxxService.java 外部服务
```

### IDDD 改进架构

经典的不一定代表最好的，也是可能存在一些问题，比如领域层和基础设施层存在相互依赖（`Repository`接口存在于领域层，其实现存在于`Infrastructure`层）。
解决方案有很多，比如将实现放在领域层，或者在应用层中实现领域层的接口，更好的方式是采用依赖倒置原则， **将重点关注在领域层** 。

![DDD_Architecture_DIP](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/18106b02284760a7cc2cc6bd5ded97dd)

采用依赖倒置原则时，事实上已经不存在分层的概念了。无论高层或低层，都只依赖与抽象，好像把整个分层架构给推平。

在架构推平的基础上，继续加入一些对称性元素，就演变成了`Alistair Cockburn`提出的[六边形架构(Hexagonal Architecture)](http://alistair.cockburn.us/Hexagonal+architecture)。

![Hexagonal_Architecture](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/080d39026600f7d330a8c9e82df905a7)

### 一些其他的领域驱动设计架构

除了上述提到的架构，也有一些其他的架构同样适用于领域驱动设计。比如`Jeffrey Palermo`提出的[洋葱架构(Onion Architecture)](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/)，`Uncle Bob`提出的[The Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)等。

![The_Onion_Architecture](https://ata2-img.oss-cn-zhangjiakou.aliyuncs.com/719541af88301437e8e3802c4587dfb0)

## TODO

后续将对模型存在的问题进行分析以及结合实际情况梳理出适合我们的架构。

+ 结合具体实例分析目前可能的问题，如何落地或改进
+ 深入了解其他更多的架构，如`Event Sourcing`和`CQRS`架构
+ 学习并了解领域驱动设计常用的框架如`Apache Isis`, `Axon`等

## 参考文献
+ [Domain-Driven Design, Eric Evans, Addison-Wesley, 2004.](http://domainlanguage.com/ddd/) - 领域驱动设计开山之作，第4-6章介绍了分层和相应的核心概念
+ [DDDSample Architecture](http://dddsample.sourceforge.net/architecture.html) - DDD一书对应的示例以及架构相应的解释
+ [Martin Fowler, Patterns of Enterprise Application Architecture](http://martinfowler.com/eaaCatalog/index.html) - Martin大叔关于企业架构的经典著作，第1章和第8章有分层架构的概述，其他各章则有对各层次具体模式的阐述
+ [Software Architecture Patterns, Mark Richards](http://www.oreilly.com/programming/free/software-architecture-patterns.csp) - 一本介绍常见架构模式的免费电子书，比较简洁
+ [Microsoft Application Architecture Guide, 2nd Edition](https://msdn.microsoft.com/en-us/library/ff650706.aspx) - 微软应用架构指南，虽然基于.NET，但不少内容和建议仍值得一看
+ [Domain Driven Design and Development In Practice](https://www.infoq.com/articles/ddd-in-practice)[领域驱动设计和开发实战](http://www.infoq.com/cn/articles/ddd-in-practice) - InfoQ上关于领域驱动设计的一些具体实践，里面的架构和现实相对接近
+ [Analysis of Software Architectures](http://www.firatatagun.com/blog/2016/01/09/analysis-of-software-architectures/)
+ [贫血模型和充血模型 from 四火的唠叨](http://www.raychase.net/238)
+ [Layers, Onions, Ports, Adapters: it's all the same by Mark Seemann](http://blog.ploeh.dk/2013/12/03/layers-onions-ports-adapters-its-all-the-same/)
+ [The Clean Architecture by Uncle Bob](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)
