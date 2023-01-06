---
title: DDD领域驱动设计
date: 2021-03-18 15:47:00
tags:
    - JAVA
categories: JAVA
---


> [代码地址](https://github.com/dinghuang/spring-cloud-framework/tree/master/dinghuang-framework-parent/dinghuang-framework-core)


# 什么是DDD
对于一个架构师来说，在软件开发中如何降低系统复杂度是一个永恒的挑战，无论是 94 年 GoF 的 Design Patterns ， 99 年的 Martin Fowler 的 Refactoring ， 02 年的 P of EAA ，还是 03 年的 Enterprise Integration Patterns ，都是通过一系列的设计模式或范例来降低一些常见的复杂度。

但是问题在于，这些书的理念是通过技术手段解决技术问题，但并没有从根本上解决业务的问题。所以 03 年 Eric Evans 的 Domain Driven Design 一书，以及后续 Vaughn Vernon 的 Implementing DDD ， Uncle Bob 的 Clean Architecture 等书，真正的从业务的角度出发，为全世界绝大部分做纯业务的开发提供了一整套的架构思路。

DDD 不是一套框架，而是一种架构思想，所以在代码层面缺乏了足够的约束，导致 DDD 在实际应用中上手门槛很高，甚至可以说绝大部分人都对 DDD 的理解有所偏差，所有认知的最终目的都是基于合理的代码结构、框架和约束，来降低 DDD 的实践门槛，提升代码质量、可测试性、安全性、健壮性。

# 好的架构设计
在做架构设计时，一个好的架构应该需要实现以下几个目标：

- 独立于框架：架构不应该依赖某个外部的库或框架，不应该被框架的结构所束缚。
- 独立于UI：前台展示的样式可能会随时发生变化（今天可能是网页、明天可能变成console、后天是独立app），但是底层架构不应该随之而变化。
- 独立于底层数据源：无论今天你用MySQL、Oracle还是MongoDB、CouchDB，甚至使用文件系统，软件架构不应该因为不同的底层数据储存方式而产生巨大改变。
- 独立于外部依赖：无论外部依赖如何变更、升级，业务的核心逻辑不应该随之而大幅变化。
可测试：无论外部依赖了什么数据库、硬件、UI或者服务，业务的逻辑应该都能够快速被验证正确性。

# 概念
## 领域模型（DOMAIN）
- 模型是对客观世界事物的一种抽象和简化。
- 它是从某个角度反映人对客观世界事物的一种认识。
- 它用于对事物的本质进行深入细致的研究。

举一个常见的例子：一个函数写了几千行，里面的if-else写了一大堆，计算各种业务规则。另一个人接手之后，分析了好几个月，才把业务逻辑彻底理清楚。从表面来看，这是代码写的不规范，要重构，把一个几千行的函数拆成一个个小的函数。从根本上来讲，就是“重要逻辑”隐藏在代码里面，没有“显性”的表达出来。这里可以引用一个观点：建模的本质就是把“重要的东西进行显性化，并进而把这些显性化的构造块，互相串联起来，组成一个体系“。


DDD通过建立一个业务域到软件域的通用模型，把问题空间同解决方案空间联系在一起，真正把领域的知识挖掘出来，让领域专家可以去驱动软件的实现。

映射概念：切分的服务。

领域就是范围。范围的重点是边界。领域的核心思想是将问题逐级细分来减低业务和系统的复杂度，这也是 DDD 讨论的核心。

如果说你要盖一栋房子，那么你需要以下的东西

有一块地
楼共有6层
每层有4个套房
那么你的领域是什么呢？

领域是房子么？可能，但是你应该知道，如果你把房子作为你的领域，那么你可能会忽略很多的小的需求。你设计的房子必须设计那里是让人住的。那么，一般而言的“房子”可能会让我们忽略一些细节，所以，我们应该缩小我们的领域，那就是“民房”。

那么，当你和建筑工人或买房子的人说起你设计的房子时，你所说的“民房”就可以很容易的被人理解了。当承包商告诉你要设计一个6层每层4套房子的时候，在语言上你是否稍微修改了一下呢？现在如果你让建筑工人到一个地方建“房子”，他们可能并没有考虑到一些“民房”的一些特性。但是如果你说了“民房”呢，他们肯定会做出合理的分析。

这就是我们将要说到的“通用语言”

### 子域
映射概念：子服务。


领域可以进一步划分成子领域，即子域。这是处理高度复杂领域的设计思想，它试图分离技术实现的复杂性。这个拆分的里面在很多架构里都有，比如 C4。

### 核心域
映射概念：核心服务。

在领域划分过程中，会不断划分子域，子域按重要程度会被划分成三类：核心域、通用域、支撑域。

决定产品核心竞争力的子域就是核心域，没有太多个性化诉求。

桃树的例子，有根、茎、叶、花、果、种子等六个子域，不同人理解的核心域不同，比如在果园里，核心域就是果是核心域，在公园里，核心域则是花。有时为了核心域的营养供应，还会剪掉通用域和支撑域（茎、叶等）。

### 通用域

映射概念：中间件服务或第三方服务。

被多个子域使用的通用功能就是通用域，没有太多企业特征，比如权限认证。


### 支撑域
映射概念：企业公共服务。

对于功能来讲是必须存在的，但它不对产品核心竞争力产生影响，也不包含通用功能，有企业特征，不具有通用性，比如数据代码类的数字字典系统。


## 统一语言（ Ubiquitous language ）
映射概念：统一概念。


关于统一语言必要性，有一个经典的通天塔故事，人类想建一座通天塔，进度很快，上帝害怕了，于是上帝让建造者说不通的语言，这样通天塔就再也没有能建起来了。统一语言是一件事情能顺利开展的基础。


由于语言上存在鸿沟，领域专家们只能模糊地描述他们想要的东西，开发人员虽然努力去理解一个自己不熟悉的领域但也只能形成模糊的认识，结果就是各说各的话，或者都是一知半解，最后到上线前才会发现漏了这个漏了那个。

定义上下文的含义。它的价值是可以解决交流障碍，不管你是 RD、PM、QA 等什么角色，让每个团队使用统一的语言（概念）来交流，甚至可读性更好的代码。

通用语言包含属于和用例场景，并且能直接反应在代码中。

可以在事件风暴（开会）中来统一语言，甚至是中英文的映射、业务与代码模型的映射等。可以使用一个表格来记录。

通用语言也并不是像，XML、Schema或Java这样的语言，它是一种自然的但经过浓缩的领域语言，它是一种开发与用户共享的语言，用来描述问题和领域模型。通用语言不是把从用户那里听到的内容翻译为开发的语言，而是为了减少误解，让用户更容易理解的草图，从而可以真正的帮助纠正错误，帮助开发获取有关的领域新知识。


统一语言体现在两个方面：

- 统一的领域术语
- 领域术语通常表示对象命名，如商品、订单等，对应实体对象

### 例子
民航业的运输统计指标为例，牵涉到与运量、运力以及周转量相关的术语，就存在 ICAO（International Civil Aviation Organization，国际民用航空组织）与IATA（International Air Transport Association，国际航空运输协会）两大体系，而中国民航局又有自己的中文解释，航空公司和各大机场亦有自己衍生的定义

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG2.png)

如果我们不明白城市对运量与航段运量的真正含义，就可能混淆这两种指标的统计计算规则。这种术语理解错误带来的缺陷往往难以发现，除非业务分析人员、开发人员与测试人员能就此知识达成一致的正确理解。

## 限界上下文（ Bounded Context ）
映射概念：服务职责划分的边界。

限界上下文，限的意思就是划分、规定，界就是界限、或者一个边界，上下文就是业务的整个流程，总的来说，可以称限界上下文为业务流程在一个划定的界限中，我们知道，业务的描述是通过通用语言来表述的，限界上下文和通用语言的关系就是：在一个特定的限界上下文只使用一套通用语言，并且保证它的清晰性和简洁性。

定义上下文的边界。领域模型存在边界之内。对于同一个概念，不同上下文会有不同的理解，比如商品，在销售阶段叫商品，在运输阶段就叫货品。理论上，限界上下文的边界就是微服务的边界，因此，理解限界上下文在设计中非常重要。

一个有界上下文可以是一个很小的程序，包括他自己的领域，自己的代码和自己的存储机制，在一个上下文里，他们应该在逻辑上一致，每个有界上下文应该独立于其他的有界上下文。

### 上下文映射图（Context Mapping）
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/0_Dig5eOh00vMkqOiw.jpg)

### 有界上下文例子

考虑一下电商系统，最初你可以认为他是一个商店上下文，但是你看的更细点，你会发现其他的上下文，例如：库存，物流，账户等。

合理的拆分一个大型的有界上下文可以让你的应用程序模块化，并且让你区分不同的关注点，而且让应用程序更易于管理和升级。每个有界上下文都有相应的责任和操作。合理的上下文划分可以让我们更容易的找到逻辑所在的位置，从而避免“大泥球”。


###  什么是“大泥球”（BBOM，Big ball of mud）
大泥球是一个不规则的、杂乱的、松散的泥巴，这类系统是典型的高度重复，快速修复，无意义增长的系统。混乱的消息传递，在一个地方几乎所有的重要信息都是全局的和重复的，所有的系统架构都可能没有被很好的定义。

我们的目标是避免所有的BBOM

我们还来说说“民房领域”吧，我们有几个有界上下文：

- 电力供应
- 停车位
- 套房
- 等等
- 
详细说一下“套房”吧，套房是由不同的房间组成，每个房间又有不同的门窗。那么现在关于屋里面的窗户就有两个问题了：

- 问题1：你可以想象出来没有屋子的窗户么？
- 问题2：如果一个窗户没有了容纳他的屋子，那么它有没有唯一的标识？

回答上面的问题会揭示出DDD中下面的概念：

- 实体（Entity）
- 值对象(Value Object)
- 聚合和聚合根(Aggreates & Aggrate root)

## 实体（ Entity ）
映射概念：Domain 或 entity。

许多对象不是由它们的属性来定义，而是通过一系列的连续性（continuity）和标识（identity）来从根本上定义的。只要一个对象在生命周期中能够保持连续性，并且独立于它的属性（即使这些属性对系统用户非常重要），那它就是一个实体。


对于实体Entity，实体核心是用唯一的标识符来定义，而不是通过属性来定义。即即使属性完全相同也可能是两个不同的对象。同时实体本身有状态的，实体又演进的生命周期，实体本身会体现出相关的业务行为，业务行为会实体属性或状态造成影响和改变。无论两个实体是多么的相似，他们都是不同的实体。

例如：

- 你家的卧室
- 博客的文章
- 博客的用户

## 值对象（ Value Objects ）
映射概念：Domain 或 entity。

当你只关心某个对象的属性时，该对象便可作为一个值对象。为其添加有意义的属性，并赋予它相应的行为。我们需要将值对象看成不变对象，不要给它任何身份标识，还应该尽量避免像实体对象一样的复杂性。

如果从值对象本身无状态，不可变，并且不分配具体的标识层面来看。那么值对象可以仅仅理解为实际的Entity对象的一个属性结合而已。该值对象附属在一个实际的实体对象上面。值对象本身不存在一个独立的生命周期，也一般不会产生独立的行为。

定义值对象的关键是值对象没有“唯一标识”。

比如 money，让它具有 id 显然是不合理的，你也不可能通过 id 查询一个 money。

定义值对象要依照具体场景的区分来看，你甚至可以把 Article 中的 Author 当成一个值对象，但一定要清楚，Author 独立存在的时候是实体，或者要拿 Author 做复杂的业务逻辑，那么 Author 也会升级为聚合根。

## 防腐层（facade）
亦称适配层。在一个上下文中，有时需要对外部上下文进行访问，通常会引入防腐层的概念来对外部上下文的访问进行一次转义。

有以下几种情况会考虑引入防腐层：

- 需要将外部上下文中的模型翻译成本上下文理解的模型。
- 不同上下文之间的团队协作关系，如果是供奉者关系，建议引入防腐层，避免外部上下文变化对本上下文的侵蚀。
- 该访问本上下文使用广泛，为了避免改动影响范围过大。


如果内部多个上下文对外部上下文需要访问，那么可以考虑将其放到通用上下文中。


## 领域服务（ Domain Services ）
领域中的一些概念不太适合建模为对象，即归类到实体对象或值对象，因为它们本质上就是一些操作，一些动作，而不是事物。这些操作或动作往往会涉及到多个领域对象，并且需要协调这些领域对象共同完成这个操作或动作。如果强行将这些操作职责分配给任何一个对象，则被分配的对象就是承担一些不该承担的职责，从而会导致对象的职责不明确很混乱。但是基于类的面向对象语言规定任何属性或行为都必须放在对象里面。所以我们需要寻找一种新的模式来表示这种跨多个对象的操作，DDD认为服务是一个很自然的范式用来对应这种跨多个对象的操作，所以就有了领域服务这个模式。和领域对象不同，领域服务是以动词开头来命名的，比如资金转帐服务可以命名为MoneyTransferService。当然，你也可以把服务理解为一个对象，但这和一般意义上的对象有些区别。因为一般的领域对象都是有状态和行为的，而领域服务没有状态只有行为。需要强调的是领域服务是无状态的，它存在的意义就是协调领域对象共完成某个操作，所有的状态还是都保存在相应的领域对象中。我觉得模型（实体）与服务（场景）是对领域的一种划分，模型关注领域的个体行为，场景关注领域的群体行为，模型关注领域的静态结构，场景关注领域的动态功能。这也符合了现实中出现的各种现象，有动有静，有独立有协作。

领域服务还有一个很重要的功能就是可以避免领域逻辑泄露到应用层。因为如果没有领域服务，那么应用层会直接调用领域对象完成本该是属于领域服务该做的操作，这样一来，领域层可能会把一部分领域知识泄露到应用层。因为应用层需要了解每个领域对象的业务功能，具有哪些信息，以及它可能会与哪些其他领域对象交互，怎么交互等一系列领域知识。因此，引入领域服务可以有效的防治领域层的逻辑泄露到应用层。对于应用层来说，从可理解的角度来讲，通过调用领域服务提供的简单易懂但意义明确的接口肯定也要比直接操纵领域对象容易的多。

说到领域服务，还需要提一下软件中一般有三种服务：应用层服务、领域服务、基础服务。

### 应用层服务
- 获取输入（如一个XML请求）；
- 发送消息给领域层服务，要求其实现转帐的业务逻辑；
- 领域层服务处理成功，则调用基础层服务发送Email通知；

### 领域层服务
- 获取源帐号和目标帐号，分别通知源帐号和目标帐号进行扣除金额和增加金额的操作；
- 提供返回结果给应用层；

### 基础层服务
按照应用层的请求，发送Email通知；

所以，从上面的例子中可以清晰的看出，每种服务的职责

## 领域事件（ Domain Events ）
领域专家所关心的发生在领域中的一些事件。
将领域中所发生的活动建模成一系列的离散事件。每个事件都用领域对象来表示。领域事件是领域模型的组成部分，表示领域中所发生的事情。

一个领域事件可以理解为是发生在一个特定领域中的事件，是你希望在同一个领域中其他部分知道并产生后续动作的事件。但是并不是所有发生过的事情都可以成为领域事件。一个领域事件必须对业务有价值，有助于形成完整的业务闭环，也即一个领域事件将导致进一步的业务操作。

领域事件可以是业务流程的一个步骤，例如订单提交，客户付费100元，订单完工等。领域事件也可以是定时发生的事情，例如每晚对账完成。或者是一个事件发生后引发的后续动作，例如客户输错密码三次后发生锁定账户的事件。

如果在通用语言中存在“当a发生时，我们就需要做到b。”这样的描述，则表明a可以定义成一个领域事件。领域事件的命名一般也就是“产生事件的对象名称+完成的动作的过去式”的形式，比如：订单已经发货的事件（OrderDispatchedEvent）、订单已被收货和确认的事件（OrderConfirmedEvent）等。

### 为什么需要领域事件
领域事件也是一种基于事件的架构（EDA）。事件架构的好处可以把处理的流程解耦，实现系统可扩展性，提高主业务流程的内聚性。


举例而言：用户提交一个订单，系统在完成订单保存后，可能还需要发送一个通知，另外可以产生一系列的后台服务的活动。如果把这一系列的动作放入一个处理过程中，会产生几个的明显问题：
一个是订单提交的的事务比较长，性能会有问题，甚至在极端情况下容易引发数据库的严重故障；另外订单提交的服务内聚性差，可维护性差，在业务流程发生变更时候，需要频繁修改主程序。


如果改为事件驱动模式，把订单提交后触发一个事件，在订单保存后，触发订单提交事件。通知和后续的各种服务动作可以通过订阅这个事件，在自己的实现空间内实现对应的逻辑，这样就把订单提交和后续其他非主要活动从订单提交业务中剥离，实现了订单提交业务高内聚和低耦合性。


领域事件也继承了事件的作用，当领域中发生了一些活动后，通过领域事件可以把这些活动所产生的副作用显式而不是隐式的表达出来，这种影响可以影响一个或多个聚合对象，而且可以获得更好的扩展性，对数据的锁影响也更小。

#### 领域事件的特点

首先是解决领域的聚合性问题。DDD中的聚合有一个原则是，在单个事务中，只允许对一个聚合对象进行修改，由此产生的其他改变必须在单独的事务中完成。如果一个业务跨多个聚合对象，领域事件会是一个不错的工具来解决这个问题。通过领域事件的方式可以达到各个组件之间的数据一致性，通过最终一致性取代事务一致性。

- 首先是解决领域的聚合性问题。DDD中的聚合有一个原则是，在单个事务中，只允许对一个聚合对象进行修改，由此产生的其他改变必须在单独的事务中完成。如果一个业务跨多个聚合对象，领域事件会是一个不错的工具来解决这个问题。通过领域事件的方式可以达到各个组件之间的数据一致性，通过最终一致性取代事务一致性。

- 其次领域事件也是一种领域分析的工具，有时从领域专家的话中，我们看不出领域事件的迹象，但是业务需求依然有可能需要领域事件。动态流的事件模型加上结合DDD的聚合实体状态和BC，可以有效进行领域建模。

领域事件可以通过观察者模式和订阅模式进行实现。比较常见的实现方式是事件总线（Event Bus）。

## 模块（ Modules ）
模块（Module）是 DDD 中明确提到的一种控制限界上下文的手段，在我们的工程中，一般尽量用一个模块来表示一个领域的限界上下文。

如代码中所示，一般的工程中包的组织方式为 {com.公司名.组织架构.业务.上下文.*}，这样的组织结构能够明确地将一个上下文限定在包的内部。

```
import com.company.team.bussiness.counter.*;//计数上下文
import com.company.team.bussiness.category.*;//分类上下文
import com.company.team.bussiness.comment.*;//评论上下文
```
对于模块内的组织结构，一般情况下我们是按照领域对象、领域服务、领域资源库、防腐层等组织方式定义的。

```
import com.company.team.bussiness.cms.domain.valobj.*;//领域对象-值对象
import com.company.team.bussiness.cms.domain.entity.*;//领域对象-实体
import com.company.team.bussiness.cms.domain.aggregate.*;//领域对象-聚合根
import com.company.team.bussiness.cms.service.*;//领域服务
import com.company.team.bussiness.cms.repo.*;//领域资源库
import com.company.team.bussiness.cms.facade.*;//领域防腐层
```

## 聚合（ Aggregate ）
映射概念：包。

聚合，它通过定义对象之间清晰的所属关系和边界来实现领域模型的内聚，并避免了错综复杂的难以维护的对象关系网的形成。聚合定义了一组具有内聚关系的相关对象的集合，我们把聚合看作是一个修改数据的单元。

### 聚合有以下一些特点：
- 每个聚合有一个根和一个边界，边界定义了一个聚合内部有哪些实体或值对象，根是聚合内的某个实体；
- 聚合内部的对象之间可以相互引用，但是聚合外部如果要访问聚合内部的对象时，必须通过聚合根开始导航，绝对不能绕过聚合根直接访问聚合内的对象，也就是说聚合根是外部可以保持 对它的引用的唯一元素；
- 聚合内除根以外的其他实体的唯一标识都是本地标识，也就是只要在聚合内部保持唯一即可，因为它们总是从属于这个聚合的；
- 聚合根负责与外部其他对象打交道并维护自己内部的业务规则；
- 基于聚合的以上概念，我们可以推论出从数据库查询时的单元也是以聚合为一个单元，也就是说我们不能直接查询聚合内部的某个非根的对象；
- 聚合内部的对象可以保持对其他聚合根的引用；
- 删除一个聚合根时必须同时删除该聚合内

## 聚合根
映射概念：包。

一个上下文内可能包含多个聚合，每个聚合都有一个根实体，叫做聚合根，一个聚合只有一个聚合根。

#### 关于如何识别聚合以及聚合根的问题：

可以先从业务的角度深入思考，然后慢慢分析出有哪些对象是：

- 有独立存在的意义，即它是不依赖于其他对象的存在它才有意义的；
- 可以被独立访问的，还是必须通过某个其他对象导航得到的；

#### 如何识别聚合
我觉得这个需要从业务的角度深入分析哪些对象它们的关系是内聚的，即我们会把他们看成是一个整体来考虑的；然后这些对象我们就可以把它们放在一个聚合内。所谓关系是内聚的，是指这些对象之间必须保持一个固定规则，固定规则是指在数据变化时必须保持不变的一致性规则。当我们在修改一个聚合时，我们必须在事务级别确保整个聚合内的所有对象满足这个固定规则。作为一条建议，聚合尽量不要太大，否则即便能够做到在事务级别保持聚合的业务规则完整性，也可能会带来一定的性能问题。有分析报告显示，通常在大部分领域模型中，有70%的聚合通常只有一个实体，即聚合根，该实体内部没有包含其他实体，只包含一些值对象；另外30%的聚合中，基本上也只包含两到三个实体。这意味着大部分的聚合都只是一个实体，该实体同时也是聚合根。

#### 如何识别聚合根
如果一个聚合只有一个实体，那么这个实体就是聚合根；如果有多个实体，那么我们可以思考聚合内哪个对象有独立存在的意义并且可以和外部直接进行交互。

上面给出的例子中：

房子、订单和问题是聚合根
窗户、订单备注和问题详情是聚合
当数据改变时，所有相关的对象被看作一个整体。

所有的外部数据访问都需要通过同一个根节点进入，这个根就是根节点

## 资源库（ Repository ）
- 仓储被设计出来的目的是基于这个原因：领域模型中的对象自从被创建出来后不会一直留在内存中活动的，当它不活动时会被持久化到数据库中，然后当需要的时候我们会重建该对象；重建对象就是根据数据库中已存储的对象的状态重新创建对象的过程；所以，可见重建对象是一个和数据库打交道的过程。从更广义的角度来理解，我们经常会像集合一样从某个类似集合的地方根据某个条件获取一个或一些对象，往集合中添加对象或移除对象。也就是说，我们需要提供一种机制，可以提供类似集合的接口来帮助我们管理对象。仓储就是基于这样的思想被设计出来的；
- 仓储里面存放的对象一定是聚合，原因是之前提到的领域模型中是以聚合的概念去划分边界的；聚合是我们更新对象的一个边界，事实上我们把整个聚合看成是一个整体概念，要么一起被取出来，要么一起被删除。我们永远不会单独对某个聚合内的子对象进行单独查询或做更新操作。因此，我们只对聚合设计仓储。
- 仓储还有一个重要的特征就是分为仓储定义部分和仓储实现部分，在领域模型中我们定义仓储的接口，而在基础设施层实现具体的仓储。这样做的原因是：由于仓储背后的实现都是在和数据库打交道，但是我们又不希望客户（如应用层）把重点放在如何从数据库获取数据的问题上，因为这样做会导致客户（应用层）代码很混乱，很可能会因此而忽略了领域模型的存在。所以我们需要提供一个简单明了的接口，供客户使用，确保客户能以最简单的方式获取领域对象，从而可以让它专心的不会被什么数据访问代码打扰的情况下协调领域对象完成业务逻辑。这种通过接口来隔离封装变化的做法其实很常见。由于客户面对的是抽象的接口并不是具体的实现，所以我们可以随时替换仓储的真实实现，这很有助于我们做单元测试。
- 尽管仓储可以像集合一样在内存中管理对象，但是仓储一般不负责事务处理。一般事务处理会交给一个叫“工作单元（Unit Of Work）”的东西。关于工作单元的详细信息我在下面的讨论中会讲到。
- 另外，仓储在设计查询接口时，可能还会用到规格模式（Specification Pattern），我见过的最厉害的规格模式应该就是LINQ以及DLINQ查询了。一般我们会根据项目中查询的灵活度要求来选择适合的仓储查询接口设计。通常情况下只需要定义简单明了的具有固定查询参数的查询接口就可以了。只有是在查询条件是动态指定的情况下才可能需要用到Specification等模式。


领域对象需要资源存储，资源库可以理解成 DAO，但它比 DAO 更宽泛，存储的手段可以是多样化的，常见的无非是数据库、分布式缓存、本地缓存等。资源库（Repository）的作用，就是对领域的存储和访问进行统一管理的对象。

在系统中，我们是通过如下的方式组织资源库的。

```
import com.company.team.bussiness.repo.dao.ArticleDao;//数据库访问对象-文章
import com.company.team.bussiness.repo.dao.CommentDao;//数据库访问对象-评论
import com.company.team.bussiness.repo.dao.po.ArticlePO;//数据库持久化对象-文章
import com.company.team.bussiness.repo.dao.po.CommentPO;//数据库持久化对象-评论
import com.company.team.bussiness.repo.cache.ArticleObj;//分布式缓存访问对象-文章缓存访问
```
资源库对外的整体访问由 Repository 提供，它聚合了各个资源库的数据信息，同时也承担了资源存储的逻辑（例如缓存更新机制等）。

# 四种 Domain 模式
除了晦涩难懂的概念外，让我们最难接受的可能就是模型的运用了，Spring 思想中，Domain 只是数据的载体，所有行为都在 Service 中使用 Domain 封装后流转，而 OOP 讲究一对象维度来执行业务，所以，DDD 中的对象是用行为的（理解这点非常重要哦）。

这里我为你总结了全部的四种领域模式，供你区分和理解：

- 失血模型
- 贫血模型
- 充血模型
- 胀血模型

## 失血模型
Domain Object 只有属性的 getter/setter 方法的纯数据类，所有的业务逻辑完全由 business object 来完成。

## 贫血模型
简单来说，就是 Domain Object 包含了不依赖于持久化的领域逻辑，而那些依赖持久化的领域逻辑被分离到 Service 层。


意这个模式不在 Domain 层里依赖 DAO。持久化的工作还需要在 DAO 或者 Service 中进行。

这样做的优缺点

优点：各层单向依赖，结构清晰。

缺点：

- Domain Object 的部分比较紧密依赖的持久化 Domain Logic 被分离到 Service 层，显得不够 OO
- Service 层过于厚重


## 充血模型
充血模型和第二种模型差不多，区别在于业务逻辑划分，将绝大多数业务逻辑放到 Domain 中，Service 是很薄的一层，封装少量业务逻辑，并且不和 DAO 打交道：

> Service (事务封装) —> Domain Object <—> DAO

所有业务逻辑都在 Domain 中，事务管理也在 Item 中实现。这样做的优缺点如下。

优点：

- 更加符合 OO 的原则；
- Service 层很薄，只充当 Facade 的角色，不和 DAO 打交道。
缺点：

- DAO 和 Domain Object 形成了双向依赖，复杂的双向依赖会导致很多潜在的问题。
- 如何划分 Service 层逻辑和 Domain 层逻辑是非常含混的，在实际项目中，由于设计和开发人员的水平差异，可能 导致整个结构的混乱无序。

## 充血模型
基于充血模型的第三个缺点，干脆取消 Service 层，只剩下 Domain Object 和 DAO 两层，在 Domain Object 的 Domain Logic 上面封装事务。

> Domain Object (事务封装，业务逻辑) <—> DAO

似乎 Ruby on rails 就是这种模型，它甚至把 Domain Object 和 DAO 都合并了。

这样做的优缺点：

- 简化了分层
- 也算符合 OO


该模型缺点：

- 很多不是 Domain Logic 的 Service 逻辑也被强行放入 Domain Object ，引起了 Domain Object 模型的不稳定；
- Domain Object 暴露给 Web 层过多的信息，可能引起意想不到的副作用。


# 总结

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/v2-32711c0238ff0ee8a15af95faf13c2c9_1440w.jpg)


# DDD分层架构
优点：
- 开发人员可以只关注整个结构中的某一层。
- 可以很容易的用新的实现来替换原有层次的实现。
- 可以降低层与层之间的依赖。
- 有利于标准化。
- 利于各层逻辑的复用。
缺点：
- 降低了系统的性能。这是显然的，因为增加了中间层，不过可以通过缓存机制来改善。
- 可能会导致级联的修改。这种修改尤其体现在自上而下的方向，不过可以通过依赖倒置来改善。

## 四层架构
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG9.png)
- User Interface为用户界面层（或表示层），负责向用户显示信息和解释用户命令。这里指的用户可以是另一个计算机系统，不一定是使用用户界面的人。
- Application为应用层，定义软件要完成的任务，并且指挥表达领域概念的对象来解决问题。这一层所负责的工作对业务来说意义重大，也是与其它系统的应用层进行交互的必要渠道。应用层要尽量简单，不包含业务规则或者知识，而只为下一层中的领域对象协调任务，分配工作，使它们互相协作。它没有反映业务情况的状态，但是却可以具有另外一种状态，为用户或程序显示某个任务的进度。
- Domain为领域层（或模型层），负责表达业务概念，业务状态信息以及业务规则。尽管保存业务状态的技术细节是由基础设施层实现的，但是反映业务情况的状态是由本层控制并且使用的。领域层是业务软件的核心，领域模型位于这一层。
- Infrastructure层为基础实施层，向其他层提供通用的技术能力：为应用层传递消息，为领域层提供持久化机制，为用户界面层绘制屏幕组件，等等。基础设施层还能够通过架构框架来支持四个层次间的交互模式。
传统的四层架构都是限定型松散分层架构，即Infrastructure层的任意上层都可以访问该层（“L”型），而其它层遵守严格分层架构

在四层架构模式的实践中，对于分层的本地化定义主要为：

- User Interface层主要是Restful消息处理，配置文件解析，等等。
- Application层主要是多进程管理及调度，多线程管理及调度，多协程调度和状态机管理，等等。
- Domain层主要是领域模型的实现，包括领域对象的确立，这些对象的生命周期管理及关系，领域服务的定义，领域事件的发布，等等。
- Infrastructure层主要是业务平台，编程框架，第三方库的封装，基础算法，等等。

> 严格意义上来说，User Interface指的是用户界面，Restful消息和配置文件解析等处理应该放在Application层，User Interface层没有的话就空缺。但User Interface也可以理解为用户接口，所以将Restful消息和配置文件解析等处理放在User Interface层也行。

## 五层架构
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG10.png)
- User Interface是用户接口层，主要用于处理用户发送的Restful请求和解析用户输入的配置文件等，并将信息传递给Application层的接口。
- Application层是应用层，负责多进程管理及调度、多线程管理及调度、多协程调度和维护业务实例的状态模型。当调度层收到用户接口层的请求后，委托Context层与本次业务相关的上下文进行处理。
- Context是环境层，以上下文为单位，将Domain层的领域对象cast成合适的role，让role交互起来完成业务逻辑。
- Domain层是领域层，定义领域模型，不仅包括领域对象及其之间关系的建模，还包括对象的角色role的显式建模。
- Infrastructure层是基础实施层，为其他层提供通用的技术能力：业务平台，编程框架，持久化机制，消息机制，第三方库的封装，通用算法，等等。


很多DDD落地实践，都是面向控制面或管理面且消息交互比较多的系统。这类系统的一次业务，包含一组同步消息或异步消息构成的序列，如果都放在Context层，会导致该层的代码比较复杂，于是我们考虑：

- Context层在面向控制面或管理面且消息交互比较多的系统中又分裂成两层，即Context层和大Context层。
- Context层处理单位为Action，对应一条同步消息或异步消息。
- 大Context层对应一个事务处理，由一个Action序列组成，一般通过Transaction DSL实现，所以我们习惯把大Context层叫做Transaction DSL层。
- Application层在面向控制面或管理面且消息交互比较多的系统中经常会做一些调度相关的工作，所以我们习惯把Application层叫做Scheduler层。


因此，在面向控制面或管理面且消息交互比较多的系统中，DDD分层架构模式就变成了六层，
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG11.png)

在实践中，将这六层的本地化定义为：

- User Interface是用户接口层，主要用于处理用户发送的Restful请求和解析用户输入的配置文件等，并将信息传递给Scheduler层的接口。
- Scheduler是调度层，负责多进程管理及调度、多线程管理及调度、多协程调度和维护业务实例的状态模型。当调度层收到用户接口层的请求后，委托Transaction层与本次操作相关的事务进行处理。
- Transaction是事务层，对应一个业务流程，比如UE Attach，将多个同步消息或异步消息的处理序列组合成一个事务，而且在大多场景下，都有选择结构。万一事务执行失败，则立即进行回滚。当事务层收到调度层的请求后，委托Context层的Action进行处理，常常还伴随使用Context层的Specification（谓词）进行Action的选择。
- Context是环境层，以Action为单位，处理一条同步消息或异步消息，将Domain层的领域对象cast成合适的role，让role交互起来完成业务逻辑。环境层通常也包括Specification的实现，即通过Domain层的知识去完成一个条件判断。
- Domain层是领域层，定义领域模型，不仅包括领域对象及其之间关系的建模，还包括对象的角色role的显式建模。
- Infrastructure层是基础实施层，为其他层提供通用的技术能力：业务平台，编程框架，持久化机制，消息机制，第三方库的封装，通用算法，等等。


事务层的核心是事务模型，事务模型的框架代码一般放在基础设施层。

综上所述，DDD六层架构可以看做是DDD五层架构在特定领域的变体，我们统称为DDD五层架构，而DDD五层架构与传统的四层架构类似，都是限定型松散分层架构。

## 六边形架构
有一种方法可以改进分层架构，即依赖倒置原则(Dependency Inversion Principle, DIP)，它通过改变不同层之间的依赖关系达到改进目的。

> 依赖倒置原则由Robert C. Martin提出，正式定义为：
高层模块不应该依赖于底层模块，两者都应该依赖于抽象。
抽象不应该依赖于细节，细节应该依赖于抽象。

根据该定义，DDD分层架构中的低层组件应该依赖于高层组件提供的接口，即无论高层还是低层都依赖于抽象，整个分层架构好像被推平了。如果我们把分层架构推平，再向其中加入一些对称性，就会出现一种具有对称性特征的架构风格，即六边形架构。六边形架构是Alistair Cockburn在2005年提出的，在这种架构中，不同的客户通过“平等”的方式与系统交互。需要新的客户吗？不是问题。只需要添加一个新的适配器将客户输入转化成能被系统API所理解的参数就行。同时，对于每种特定的输出，都有一个新建的适配器负责完成相应的转化功能。

六边形架构也称为端口与适配器，如下图所示：
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/8906859-c7a0c865b8217865.png)

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/8906859-27500c1a61ca00ec.png)

六边形每条不同的边代表了不同类型的端口，端口要么处理输入，要么处理输出。对于每种外界类型，都有一个适配器与之对应，外界通过应用层API与内部进行交互。上图中有3个客户请求均抵达相同的输入端口（适配器A、B和C），另一个客户请求使用了适配器D。假设前3个请求使用了HTTP协议（浏览器、REST和SOAP等），而后一个请求使用了AMQP协议（比如RabbitMQ）。端口并没有明确的定义，它是一个非常灵活的概念。无论采用哪种方式对端口进行划分，当客户请求到达时，都应该有相应的适配器对输入进行转化，然后端口将调用应用程序的某个操作或者向应用程序发送一个事件，控制权由此交给内部区域。
应用程序通过公共API接收客户请求，使用领域模型来处理请求。我们可以将DDD战术设计的建模元素Repository的实现看作是持久化适配器，该适配器用于访问先前存储的聚合实例或者保存新的聚合实例。正如图中的适配器E、F和G所展示的，我们可以通过不同的方式实现资源库，比如关系型数据库、基于文档的存储、分布式缓存或内存存储等。如果应用程序向外界发送领域事件消息，我们将使用适配器H进行处理。该适配器处理消息输出，而上面提到的处理AMQP消息的适配器则是处理消息输入的，因此应该使用不同的端口。

我们在实际的项目开发中，不同层的组件可以同时开发。当一个组件的功能明确后，就可以立即启动开发。由于该组件的用户有多个，并且这些用户的侧重点不同，所以需要提供多个不同的接口。同时，这些用户的认识也是不断深入的，可能会多次重构相关的接口。于是，组件的多个用户经常会找组件的开发者讨论这些问题，无形中降低了组件的开发效率。
我们换一种方式，组件的开发者在明确了组件的功能后就专注于功能的开发，确保功能稳定和高效。组件的用户自己定义组件的接口（端口），然后基于接口写测试，并不断演进接口。在跨层集成测试时，由组件开发者或用户再开发一个适配器就可以了。

### 六边形架构模式的演变
尽管六边形架构模式已经很好，但是没有最好只有更好，演变没有尽头。在六边形架构模式提出后的这些年，又依次衍生出三种六边形架构模式的变体，感兴趣的读者可以点击链接自行学习：

- Jeffrey Palermo在2008年提出了洋葱架构，六边形架构是洋葱架构的一个超集。
- Robert C. Martin在2012年提出了干净架构（Clean Architecture），这是六边形架构的一个变体。
- Russ Miles在2013年提出了Life Preserver设计，这是一种基于六边形架构的设计。

# DDD实践
假如你是一个团队 Leader 或者架构师，当你接手一个旧系统维护及重构的任务时，你该如何改造呢？是否觉得哪里都不对但由于业务认知的不熟悉而无从下手呢？


- 通过公共平台大概梳理出系统之间的调用关系（一般中等以上公司都具备 RPC 和 HTTP 调用关系，无脑的挨个系统查询即可），画出来的可能会很乱，也可能会比较清晰，但这就是现状。
- 分配组员每个人认领几个项目，来梳理项目维度关系，这些关系包括：对外接口、交互、用例、MQ 等的详细说明。个别核心系统可以画出内部实体或者聚合根。
- 小组开会，挨个 review 每个系统的业务概念，达到组内统一语言。
- 根据以上资料，即可看出哪些不合理的调用关系（比如循环调用、不规范的调用等），甚至不合理的分层。
- 根据主线业务自顶向下细分领域，以及限界上下文。此过程可能会颠覆之前的系统划分。
- 根据业务复杂性，指定领域模型，选择贫血或者充血模型。团队内部最好实行统一习惯，以免出现交接成本过大。
- 分工进行开发，并设置 deadline，注意，不要单一的设置一个 deadline，要设置中间 check 时间，比如 dealline 是 1 月 20 日，还要设置两个 check 时间，分别沟通代码风格及边界职责，以免 deadline 时延期。

### 事件风暴（Event Storming）

事件风暴也被称为事件建模，形式有点类似于头脑风暴的方法,通过事件风暴的方法可以快速分析复杂业务领域，完成领域建模的目标。

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG3.png)

> 事件风暴是一项团队活动，旨在通过领域事件识别出聚合根，进而划分微服务的限界上下文。在活动中，团队先通过头脑风暴的形式罗列出领域中所有的领域事件，整合之后形成最终的领域事件集合，然后对于每一个事件，标注出导致该事件的命令（Command），再然后为每个事件标注出命令发起方的角色，命令可以是用户发起，也可以是第三方系统调用或者是定时器触发等。最后对事件进行分类整理出聚合根以及限界上下文。

事件风暴方法通常有以下几个步骤：

- 邀请合适的人参加
- 提供无限制的建模空间
- 发掘领域事件，可使用橙色贴纸标识
- 发掘领域事件的来源，探询命令，可使用蓝色贴纸标识。多种来源也可以- 考虑用不同颜色来区分，例如用黄色表示角色，粉色表示外部系统，红色表示时间触发。
- 寻找聚合，可用绿色贴纸标识
- 持续探索，发掘子域，BC，用户角色，验收测试标准，补充信息等。

事件风暴的步骤可以如下图所示：
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG4.png)

在我们的一次产品的重构活动中也采用了事件风暴方法。系统代码维护了10几年，代码中存在大量的“坏味道”：重复代码，过长函数，过大的类，过长的参数列表，发散式变化，霰弹式修改，镀金问题，注释不清等问题。实际研发过程中也是经常出现一点改动都可能会引起不可预测的结果，重构势在必行。

但是在重构过程中，也没有人可以说清楚现有系统的逻辑，如何重构成为了一个难题。重构过程我们引入了咨询公司给我们的方法，采用了事件风暴的办法，通过对领域中所发生的事情（也就是领域事件）来探索这个领域，并且使用便签来描述领域中的事件，这些便签会沿着时间轴贴到一个很大的建模面板上。

举例来说，能够引发事件的事情包括用户行为、外部系统所发生的事情以及时间的流逝。事件也有助于找到领域的边界，对术语的不同阐述可能就意味着存在边界。

准备工作，四色贴纸：
- 橙色：事件，某个动作的结果，以“XX已XX”的方式表示，比如“用户信息已查询”
- 蓝色：属性，事件相关的输入、输出数据等
- 黄色：命令，某个动作，比如“查找用户信息”
- 绿色：实体，命令的触发者
- 开始梳理业务，将结果贴到白版上
- 继续深入梳理，将整个过程的模型、关键数据等梳理出来，贴在白板上
确定重构指导思路，执行重构动作，重构的同时引入单元测试保障重构的质量

# 实践
我们先看一个简单的例子，这个 case 的业务逻辑如下：

> 一个新应用在全国通过 地推业务员 做推广，需要做一个用户注册系统，同时希望在用户注册后能够通过用户电话（先假设仅限座机）的地域（区号）对业务员发奖金。

先不要去纠结这个根据用户电话去发奖金的业务逻辑是否合理，也先不要去管用户是否应该在注册时和业务员做绑定，这里我们看的主要还是如何更加合理的去实现这个逻辑。一个简单的用户和用户注册的代码实现如下：

```
public class User {
    Long userId;
    String name;
    String phone;
    String address;
    Long repId;
}

public class RegistrationServiceImpl implements RegistrationService {

    private SalesRepRepository salesRepRepo;
    private UserRepository userRepo;

    public User register(String name, String phone, String address) 
      throws ValidationException {
        // 校验逻辑
        if (name == null || name.length() == 0) {
            throw new ValidationException("name");
        }
        if (phone == null || !isValidPhoneNumber(phone)) {
            throw new ValidationException("phone");
        }
        // 此处省略address的校验逻辑

        // 取电话号里的区号，然后通过区号找到区域内的SalesRep
        String areaCode = null;
        String[] areas = new String[]{"0571", "021", "010"};
        for (int i = 0; i < phone.length(); i++) {
            String prefix = phone.substring(0, i);
            if (Arrays.asList(areas).contains(prefix)) {
                areaCode = prefix;
                break;
            }
        }
        SalesRep rep = salesRepRepo.findRep(areaCode);

        // 最后创建用户，落盘，然后返回
        User user = new User();
        user.name = name;
        user.phone = phone;
        user.address = address;
        if (rep != null) {
            user.repId = rep.repId;
        }

        return userRepo.save(user);
    }

    private boolean isValidPhoneNumber(String phone) {
        String pattern = "^0[1-9]{2,3}-?\\d{8}$";
        return phone.matches(pattern);
    }
}
```

我们日常绝大部分代码和模型其实都跟这个是类似的，乍一看貌似没啥问题，但我们再深入一步，从以下四个维度去分析一下：接口的清晰度（可阅读性）、数据验证和错误处理、业务逻辑代码的清晰度、和可测试性。

## 问题1 - 接口的清晰度
在Java代码中，对于一个方法来说所有的参数名在编译时丢失，留下的仅仅是一个参数类型的列表，所以我们重新看一下以上的接口定义，其实在运行时仅仅是：

```
User register(String, String, String);
```
所以以下的代码是一段编译器完全不会报错的，很难通过看代码就能发现的 bug ：

```
service.register("殷浩", "浙江省杭州市余杭区文三西路969号", "0571-12345678");
```
当然，在真实代码中运行时会报错，但这种 bug 是在运行时被发现的，而不是在编译时。普通的 Code Review 也很难发现这种问题，很有可能是代码上线后才会被暴露出来。这里的思考是，有没有办法在编码时就避免这种可能会出现的问题？

另外一种常见的，特别是在查询服务中容易出现的例子如下：

```
User findByName(String name);
User findByPhone(String phone);
User findByNameAndPhone(String name, String phone);
```
在这个场景下，由于入参都是 String 类型，不得不在方法名上面加上 ByXXX 来区分，而 findByNameAndPhone 同样也会陷入前面的入参顺序错误的问题，而且和前面的入参不同，这里参数顺序如果输错了，方法不会报错只会返回 null，而这种 bug 更加难被发现。这里的思考是，有没有办法让方法入参一目了然，避免入参错误导致的 bug ？

## 问题2 - 数据验证和错误处理

在前面这段数据校验代码：
```
if (phone == null || !isValidPhoneNumber(phone)) {
    throw new ValidationException("phone");
}
```
在日常编码中经常会出现，一般来说这种代码需要出现在方法的最前端，确保能够 fail-fast 。但是假设你有多个类似的接口和类似的入参，在每个方法里这段逻辑会被重复。而更严重的是如果未来我们要拓展电话号去包含手机时，很可能需要加入以下代码：


```
if (phone == null || !isValidPhoneNumber(phone) || !isValidCellNumber(phone)) {
    throw new ValidationException("phone");
}
```
如果你有很多个地方用到了 phone 这个入参，但是有个地方忘记修改了，会造成 bug 。这是一个 DRY 原则被违背时经常会发生的问题。

如果有个新的需求，需要把入参错误的原因返回，那么这段代码就变得更加复杂：

```
if (phone == null) {
    throw new ValidationException("phone不能为空");
} else if (!isValidPhoneNumber(phone)) {
    throw new ValidationException("phone格式错误");
}
```
可以想像得到，代码里充斥着大量的类似代码块时，维护成本要有多高。

最后，在这个业务方法里，会（隐性或显性的）抛 ValidationException，所以需要外部调用方去try/catch，而业务逻辑异常和数据校验异常被混在了一起，是否是合理的？

在传统Java架构里有几个办法能够去解决一部分问题，常见的如BeanValidation注解或ValidationUtils类，比如：
```
// Use Bean Validation
User registerWithBeanValidation(
  @NotNull @NotBlank String name,
  @NotNull @Pattern(regexp = "^0?[1-9]{2,3}-?\\d{8}$") String phone,
  @NotNull String address
);

// Use ValidationUtils:
public User registerWithUtils(String name, String phone, String address) {
    ValidationUtils.validateName(name); // throws ValidationException
    ValidationUtils.validatePhone(phone);
    ValidationUtils.validateAddress(address);
    ...
}
```

但这几个传统的方法同样有问题，

BeanValidation：

- 通常只能解决简单的校验逻辑，复杂的校验逻辑一样要写代码实现定制校验器
- 在添加了新校验逻辑时，同样会出现在某些地方忘记添加一个注解的情况，DRY原则还是会被违背


ValidationUtils类：

- 当大量的校验逻辑集中在一个类里之后，违背了Single Responsibility单一性原则，导致代码混乱和不可维护
- 业务异常和校验异常还是会混杂

所以，有没有一种方法，能够一劳永逸的解决所有校验的问题以及降低后续的维护成本和异常处理成本呢？

## 问题3 - 业务代码的清晰度

在这段代码里：

```
String areaCode = null;
String[] areas = new String[]{"0571", "021", "010"};
for (int i = 0; i < phone.length(); i++) {
    String prefix = phone.substring(0, i);
    if (Arrays.asList(areas).contains(prefix)) {
        areaCode = prefix;
        break;
    }
}
SalesRep rep = salesRepRepo.findRep(areaCode);
```
实际上出现了另外一种常见的情况，那就是从一些入参里抽取一部分数据，然后调用一个外部依赖获取更多的数据，然后通常从新的数据中再抽取部分数据用作其他的作用。这种代码通常被称作“胶水代码”，其本质是由于外部依赖的服务的入参并不符合我们原始的入参导致的。比如，如果SalesRepRepository包含一个findRepByPhone的方法，则上面大部分的代码都不必要了。

所以，一个常见的办法是将这段代码抽离出来，变成独立的一个或多个方法：

```
private static String findAreaCode(String phone) {
    for (int i = 0; i < phone.length(); i++) {
        String prefix = phone.substring(0, i);
        if (isAreaCode(prefix)) {
            return prefix;
        }
    }
    return null;
}

private static boolean isAreaCode(String prefix) {
    String[] areas = new String[]{"0571", "021"};
    return Arrays.asList(areas).contains(prefix);
}
```
然后原始代码变为：

```
String areaCode = findAreaCode(phone);
SalesRep rep = salesRepRepo.findRep(areaCode);
```
而为了复用以上的方法，可能会抽离出一个静态工具类 PhoneUtils 。但是这里要思考的是，静态工具类是否是最好的实现方式呢？当你的项目里充斥着大量的静态工具类，业务代码散在多个文件当中时，你是否还能找到核心的业务逻辑呢？

## 问题4 - 可测试性
为了保证代码质量，每个方法里的每个入参的每个可能出现的条件都要有 TC 覆盖（假设我们先不去测试内部业务逻辑），所以在我们这个方法里需要以下的 TC ：


假如一个方法有 N 个参数，每个参数有 M 个校验逻辑，至少要有 N * M 个 TC 。

如果这时候在该方法中加入一个新的入参字段 fax ，即使 fax 和 phone 的校验逻辑完全一致，为了保证 TC 覆盖率，也一样需要 M 个新的 TC 。

而假设有 P 个方法中都用到了 phone 这个字段，这 P 个方法都需要对该字段进行测试，也就是说整体需要：

P * N * M

个测试用例才能完全覆盖所有数据验证的问题，在日常项目中，这个测试的成本非常之高，导致大量的代码没被覆盖到。而没被测试覆盖到的代码才是最有可能出现问题的地方。

在这个情况下，降低测试成本 == 提升代码质量，如何能够降低测试的成本呢？

## 解决方案

我们回头先重新看一下原始的 use case，并且标注其中可能重要的概念：

一个新应用在全国通过 地推业务员 做推广，需要做一个用户的注册系统，在用户注册后能够通过用户电话号的区号对业务员发奖金。

在分析了 use case 后，发现其中地推业务员、用户本身自带 ID 属性，属于 Entity（实体），而注册系统属于 Application Service（应用服务），这几个概念已经有存在。但是发现电话号这个概念却完全被隐藏到了代码之中。我们可以问一下自己，取电话号的区号的逻辑是否属于用户（用户的区号？）？是否属于注册服务（注册的区号？）？如果都不是很贴切，那就说明这个逻辑应该属于一个独立的概念。所以这里引入我们第一个原则：

Make Implicit Concepts Explicit

### 将隐性的概念显性化
在这里，我们可以看到，原来电话号仅仅是用户的一个参数，属于隐形概念，但实际上电话号的区号才是真正的业务逻辑，而我们需要将电话号的概念显性化，通过写一个Value Object：

```
public class PhoneNumber {
  
    private final String number;
    public String getNumber() {
        return number;
    }

    public PhoneNumber(String number) {
        if (number == null) {
            throw new ValidationException("number不能为空");
        } else if (isValid(number)) {
            throw new ValidationException("number格式错误");
        }
        this.number = number;
    }

    public String getAreaCode() {
        for (int i = 0; i < number.length(); i++) {
            String prefix = number.substring(0, i);
            if (isAreaCode(prefix)) {
                return prefix;
            }
        }
        return null;
    }

    private static boolean isAreaCode(String prefix) {
        String[] areas = new String[]{"0571", "021", "010"};
        return Arrays.asList(areas).contains(prefix);
    }

    public static boolean isValid(String number) {
        String pattern = "^0?[1-9]{2,3}-?\\d{8}$";
        return number.matches(pattern);
    }

}
```
这里面有几个很重要的元素：

- 通过 private final String number 确保 PhoneNumber 是一个（Immutable）Value Object。（一般来说 VO 都是 Immutable 的，这里只是重点强调一下）
- 校验逻辑都放在了 constructor 里面，确保只要 PhoneNumber 类被创建出来后，一定是校验通过的。
- 之前的 findAreaCode 方法变成了 PhoneNumber 类里的 getAreaCode ，突出了 areaCode 是 PhoneNumber 的一个计算属性。

这样做完之后，我们发现把 PhoneNumber 显性化之后，其实是生成了一个 Type（数据类型）和一个 Class（类）：

- Type 指我们在今后的代码里可以通过 PhoneNumber 去显性的标识电话号这个概念
- Class 指我们可以把所有跟电话号相关的逻辑完整的收集到一个文件里
这两个概念加起来，构造成了本文标题的 Domain Primitive（DP）。

我们看一下全面使用了 DP 之后效果：
```
public class User {
    UserId userId;
    Name name;
    PhoneNumber phone;
    Address address;
    RepId repId;
}

public User register(
  @NotNull Name name,
  @NotNull PhoneNumber phone,
  @NotNull Address address
) {
    // 找到区域内的SalesRep
    SalesRep rep = salesRepRepo.findRep(phone.getAreaCode());

    // 最后创建用户，落盘，然后返回，这部分代码实际上也能用Builder解决
    User user = new User();
    user.name = name;
    user.phone = phone;
    user.address = address;
    if (rep != null) {
        user.repId = rep.repId;
    }

    return userRepo.saveUser(user);
}
```

我们可以看到在使用了 DP 之后，所有的数据验证逻辑和非业务流程的逻辑都消失了，剩下都是核心业务逻辑，可以一目了然。我们重新用上面的四个维度评估一下：

## 评估1 - 接口的清晰度
重构后的方法签名变成了很清晰的：

```
public User register(Name, PhoneNumber, Address)
```
而之前容易出现的bug，如果按照现在的写法

```
service.register(new Name("殷浩"), new Address("浙江省杭州市余杭区文三西路969号"), new PhoneNumber("0571-12345678"));
```
让接口 API 变得很干净，易拓展。

## 评估2 - 数据验证和错误处理
```
public User register(
  @NotNull Name name,
  @NotNull PhoneNumber phone,
  @NotNull Address address
) // no throws
```

如前文代码展示的，重构后的方法里，完全没有了任何数据验证的逻辑，也不会抛 ValidationException 。原因是因为 DP 的特性，只要是能够带到入参里的一定是正确的或 null（Bean Validation 或 lombok 的注解能解决 null 的问题）。所以我们把数据验证的工作量前置到了调用方，而调用方本来就是应该提供合法数据的，所以更加合适。

再展开来看，使用DP的另一个好处就是代码遵循了 DRY 原则和单一性原则，如果未来需要修改 PhoneNumber 的校验逻辑，只需要在一个文件里修改即可，所有使用到了 PhoneNumber 的地方都会生效。

## 评估3 - 业务代码的清晰度
```
SalesRep rep = salesRepRepo.findRep(phone.getAreaCode());
User user = xxx;
return userRepo.save(user);
```
除了在业务方法里不需要校验数据之外，原来的一段胶水代码 findAreaCode 被改为了 PhoneNumber 类的一个计算属性 getAreaCode ，让代码清晰度大大提升。而且胶水代码通常都不可复用，但是使用了 DP 后，变成了可复用、可测试的代码。我们能看到，在刨除了数据验证代码、胶水代码之后，剩下的都是核心业务逻辑。（ Entity 相关的重构在后面文章会谈到，这次先忽略）

## 评估4 - 可测试性

当我们将 PhoneNumber 抽取出来之后，在来看测试的 TC ：

首先 PhoneNumber 本身还是需要 M 个测试用例，但是由于我们只需要测试单一对象，每个用例的代码量会大大降低，维护成本降低。
每个方法里的每个参数，现在只需要覆盖为 null 的情况就可以了，其他的 case 不可能发生（因为只要不是 null 就一定是合法的）
所以，单个方法的 TC 从原来的 N * M 变成了今天的 N + M 。同样的，多个方法的 TC 数量变成了

N + M + P

这个数量一般来说要远低于原来的数量 N* M * P ，让测试成本极大的降低。

## 评估总结

## 进阶使用

在上文我介绍了 DP 的第一个原则：将隐性的概念显性化。在这里我将介绍 DP 的另外两个原则，用一个新的案例。

## ▍案例1 - 转账
假设现在要实现一个功能，让A用户可以支付 x 元给用户 B ，可能的实现如下：

```
public void pay(BigDecimal money, Long recipientId) {
    BankService.transfer(money, "CNY", recipientId);
}
```

如果这个是境内转账，并且境内的货币永远不变，该方法貌似没啥问题，但如果有一天货币变更了（比如欧元区曾经出现的问题），或者我们需要做跨境转账，该方法是明显的 bug ，因为 money 对应的货币不一定是 CNY 。

在这个 case 里，当我们说“支付 x 元”时，除了 x 本身的数字之外，实际上是有一个隐含的概念那就是货币“元”。但是在原始的入参里，之所以只用了 BigDecimal 的原因是我们认为 CNY 货币是默认的，是一个隐含的条件，但是在我们写代码时，需要把所有隐性的条件显性化，而这些条件整体组成当前的上下文。所以 DP 的第二个原则是：


Make Implicit Context Explicit

将 隐性的 上下文 显性化

所以当我们做这个支付功能时，实际上需要的一个入参是支付金额 + 支付货币。我们可以把这两个概念组合成为一个独立的完整概念：Money。

```
@Value
public class Money {
    private BigDecimal amount;
    private Currency currency;
    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
}
```
而原有的代码则变为：

```
public void pay(Money money, Long recipientId) {
    BankService.transfer(money, recipientId);
}
```
通过将默认货币这个隐性的上下文概念显性化，并且和金额合并为 Money ，我们可以避免很多当前看不出来，但未来可能会暴雷的bug。

## 案例2 - 跨境转账

前面的案例升级一下，假设用户可能要做跨境转账从 CNY 到 USD ，并且货币汇率随时在波动：


```
public void pay(Money money, Currency targetCurrency, Long recipientId) {
    if (money.getCurrency().equals(targetCurrency)) {
        BankService.transfer(money, recipientId);
    } else {
        BigDecimal rate = ExchangeService.getRate(money.getCurrency(), targetCurrency);
        BigDecimal targetAmount = money.getAmount().multiply(new BigDecimal(rate));
        Money targetMoney = new Money(targetAmount, targetCurrency);
        BankService.transfer(targetMoney, recipientId);
    }
}
```
在这个case里，由于 targetCurrency 不一定和 money 的 Curreny 一致，需要调用一个服务去取汇率，然后做计算。最后用计算后的结果做转账。

这个case最大的问题在于，金额的计算被包含在了支付的服务中，涉及到的对象也有2个 Currency ，2 个 Money ，1 个 BigDecimal ，总共 5 个对象。这种涉及到多个对象的业务逻辑，需要用 DP 包装掉，所以这里引出 DP 的第三个原则：

Encapsulate Multi-Object Behavior

封装 多对象 行为

在这个 case 里，可以将转换汇率的功能，封装到一个叫做 ExchangeRate 的 DP 里：

```
@Value
public class ExchangeRate {
    private BigDecimal rate;
    private Currency from;
    private Currency to;

    public ExchangeRate(BigDecimal rate, Currency from, Currency to) {
        this.rate = rate;
        this.from = from;
        this.to = to;
    }

    public Money exchange(Money fromMoney) {
        notNull(fromMoney);
        isTrue(this.from.equals(fromMoney.getCurrency()));
        BigDecimal targetAmount = fromMoney.getAmount().multiply(rate);
        return new Money(targetAmount, to);
    }
}
```
ExchangeRate汇率对象，通过封装金额计算逻辑以及各种校验逻辑，让原始代码变得极其简单：
```
public void pay(Money money, Currency targetCurrency, Long recipientId) {
    ExchangeRate rate = ExchangeService.getRate(money.getCurrency(), targetCurrency);
    Money targetMoney = rate.exchange(money);
    BankService.transfer(targetMoney, recipientId);
}
```

## 讨论和总结
### ▍Domain Primitive 的定义

让我们重新来定义一下 Domain Primitive ：Domain Primitive 是一个在特定领域里，拥有精准定义的、可自我验证的、拥有行为的 Value Object 。

DP是一个传统意义上的Value Object，拥有Immutable的特性
DP是一个完整的概念整体，拥有精准定义
DP使用业务域中的原生语言
DP可以是业务域的最小组成部分、也可以构建复杂组合
注：Domain Primitive的概念和命名来自于Dan Bergh Johnsson & Daniel Deogun的书 Secure by Design。

### ▍使用 Domain Primitive 的三原则
让隐性的概念显性化
让隐性的上下文显性化
封装多对象行为

### ▍Domain Primitive 和 DDD 里 Value Object 的区别
在 DDD 中， Value Object 这个概念其实已经存在：

在 Evans 的 DDD 蓝皮书中，Value Object 更多的是一个非 Entity 的值对象
在Vernon的IDDD红皮书中，作者更多的关注了Value Object的Immutability、Equals方法、Factory方法等
Domain Primitive 是 Value Object 的进阶版，在原始 VO 的基础上要求每个 DP 拥有概念的整体，而不仅仅是值对象。在 VO 的 Immutable 基础上增加了 Validity 和行为。当然同样的要求无副作用（side-effect free）。

### ▍Domain Primitive 和 Data Transfer Object (DTO) 的区别
在日常开发中经常会碰到的另一个数据结构是 DTO ，比如方法的入参和出参。DP 和 DTO 的区别如下：


### ▍什么情况下应该用 Domain Primitive
常见的 DP 的使用场景包括：

有格式限制的 String：比如Name，PhoneNumber，OrderNumber，ZipCode，Address等
有限制的Integer：比如OrderId（>0），Percentage（0-100%），Quantity（>=0）等
可枚举的 int ：比如 Status（一般不用Enum因为反序列化问题）
Double 或 BigDecimal：一般用到的 Double 或 BigDecimal 都是有业务含义的，比如 Temperature、Money、Amount、ExchangeRate、Rating 等
复杂的数据结构：比如 Map<String, List<Integer>> 等，尽量能把 Map 的所有操作包装掉，仅暴露必要行为


## 实战 - 老应用重构的流程
在新应用中使用 DP 是比较简单的，但在老应用中使用 DP 是可以遵循以下流程按部就班的升级。在此用本文的第一个 case 为例。

### ▍第一步 - 创建 Domain Primitive，收集所有 DP 行为
在前文中，我们发现取电话号的区号这个是一个可以独立出来的、可以放入 PhoneNumber 这个 Class 的逻辑。类似的，在真实的项目中，以前散落在各个服务或工具类里面的代码，可以都抽出来放在 DP 里，成为 DP 自己的行为或属性。这里面的原则是：所有抽离出来的方法要做到无状态，比如原来是 static 的方法。如果原来的方法有状态变更，需要将改变状态的部分和不改状态的部分分离，然后将无状态的部分融入 DP 。因为 DP 本身不能带状态，所以一切需要改变状态的代码都不属于 DP 的范畴。

(代码参考 PhoneNumber 的代码，这里不再重复)

### ▍第二步 - 替换数据校验和无状态逻辑
为了保障现有方法的兼容性，在第二步不会去修改接口的签名，而是通过代码替换原有的校验逻辑和根 DP 相关的业务逻辑。比如：

```
public User register(String name, String phone, String address)
        throws ValidationException {
    if (name == null || name.length() == 0) {
        throw new ValidationException("name");
    }
    if (phone == null || !isValidPhoneNumber(phone)) {
        throw new ValidationException("phone");
    }
    
    String areaCode = null;
    String[] areas = new String[]{"0571", "021", "010"};
    for (int i = 0; i < phone.length(); i++) {
        String prefix = phone.substring(0, i);
        if (Arrays.asList(areas).contains(prefix)) {
            areaCode = prefix;
            break;
        }
    }
    SalesRep rep = salesRepRepo.findRep(areaCode);
    // 其他代码...
}
```
通过 DP 替换代码后：

```
public User register(String name, String phone, String address)
        throws ValidationException {
    
    Name _name = new Name(name);
    PhoneNumber _phone = new PhoneNumber(phone);
    Address _address = new Address(address);
    
    SalesRep rep = salesRepRepo.findRep(_phone.getAreaCode());
    // 其他代码...
}
```
通过 new PhoneNumber(phone) 这种代码，替代了原有的校验代码。

通过 _phone.getAreaCode() 替换了原有的无状态的业务逻辑。

▍第三步 - 创建新接口
创建新接口，将DP的代码提升到接口参数层：

```
public User register(Name name, PhoneNumber phone, Address address) {
    SalesRep rep = salesRepRepo.findRep(phone.getAreaCode());
}
```
▍第四步 - 修改外部调用

外部调用方需要修改调用链路，比如：

```
service.register("殷浩", "0571-12345678", "浙江省杭州市余杭区文三西路969号");
改为：

service.register(new Name("殷浩"), new PhoneNumber("0571-12345678"), new Address("浙江省杭州市余杭区文三西路969号"));
```
通过以上 4 步，就能让你的代码变得更加简洁、优雅、健壮、安全。你还在等什么？今天就去尝试吧！


# 应用架构设计

设计原则

- 单一性原则（Single Responsibility Principle）：单一性原则要求一个对象/类应该只有一个变更的原因。但是在这个案例里，代码可能会因为任意一个外部依赖或计算逻辑的改变而改变。
- 依赖反转原则（Dependency Inversion Principle）：依赖反转原则要求在代码中依赖抽象，而不是具体的实现。在这个案例里外部依赖都是具体的实现，比如YahooForexService虽然是一个接口类，但是它对应的是依赖了Yahoo提供的具体服务，所以也算是依赖了实现。同样的KafkaTemplate、MyBatis的DAO实现都属于具体实现。
- 开放封闭原则（Open Closed Principle）：开放封闭原则指开放扩展，但是封闭修改。在这个案例里的金额计算属于可能会被修改的代码，这个时候该逻辑应该需要被包装成为不可修改的计算类，新功能通过计算类的拓展实现。


## 需求背景

我们先看一个简单的案例需求如下：

用户可以通过银行网页转账给另一个账号，支持跨币种转账。

同时因为监管和对账需求，需要记录本次转账活动。

拿到这个需求之后，一个开发可能会经历一些技术选型，最终可能拆解需求如下：



1、从MySql数据库中找到转出和转入的账户，选择用 MyBatis 的 mapper 实现 DAO；2、从 Yahoo（或其他渠道）提供的汇率服务获取转账的汇率信息（底层是 http 开放接口）；

3、计算需要转出的金额，确保账户有足够余额，并且没超出每日转账上限；

4、实现转入和转出操作，扣除手续费，保存数据库；

5、发送 Kafka 审计消息，以便审计和对账用；

而一个简单的代码实现如下：
```
public class TransferController {

    private TransferService transferService;

    public Result<Boolean> transfer(String targetAccountNumber, BigDecimal amount, HttpSession session) {
        Long userId = (Long) session.getAttribute("userId");
        return transferService.transfer(userId, targetAccountNumber, amount, "CNY");
    }
}

public class TransferServiceImpl implements TransferService {

    private static final String TOPIC_AUDIT_LOG = "TOPIC_AUDIT_LOG";
    private AccountMapper accountDAO;
    private KafkaTemplate<String, String> kafkaTemplate;
    private YahooForexService yahooForex;

    @Override
    public Result<Boolean> transfer(Long sourceUserId, String targetAccountNumber, BigDecimal targetAmount, String targetCurrency) {
        // 1. 从数据库读取数据，忽略所有校验逻辑如账号是否存在等
        AccountDO sourceAccountDO = accountDAO.selectByUserId(sourceUserId);
        AccountDO targetAccountDO = accountDAO.selectByAccountNumber(targetAccountNumber);

        // 2. 业务参数校验
        if (!targetAccountDO.getCurrency().equals(targetCurrency)) {
            throw new InvalidCurrencyException();
        }

        // 3. 获取外部数据，并且包含一定的业务逻辑
        // exchange rate = 1 source currency = X target currency
        BigDecimal exchangeRate = BigDecimal.ONE;
        if (sourceAccountDO.getCurrency().equals(targetCurrency)) {
            exchangeRate = yahooForex.getExchangeRate(sourceAccountDO.getCurrency(), targetCurrency);
        }
        BigDecimal sourceAmount = targetAmount.divide(exchangeRate, RoundingMode.DOWN);

        // 4. 业务参数校验
        if (sourceAccountDO.getAvailable().compareTo(sourceAmount) < 0) {
            throw new InsufficientFundsException();
        }

        if (sourceAccountDO.getDailyLimit().compareTo(sourceAmount) < 0) {
            throw new DailyLimitExceededException();
        }

        // 5. 计算新值，并且更新字段
        BigDecimal newSource = sourceAccountDO.getAvailable().subtract(sourceAmount);
        BigDecimal newTarget = targetAccountDO.getAvailable().add(targetAmount);
        sourceAccountDO.setAvailable(newSource);
        targetAccountDO.setAvailable(newTarget);

        // 6. 更新到数据库
        accountDAO.update(sourceAccountDO);
        accountDAO.update(targetAccountDO);

        // 7. 发送审计消息
        String message = sourceUserId + "," + targetAccountNumber + "," + targetAmount + "," + targetCurrency;
        kafkaTemplate.send(TOPIC_AUDIT_LOG, message);

        return Result.success(true);
    }

}
```

## 流程图

在重构之前，我们先画一张流程图，描述当前代码在做的每个步骤：
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/DDD%E9%A2%86%E5%9F%9F%E9%A9%B1%E5%8A%A8%E8%AE%BE%E8%AE%A1.jpeg)

这是一个传统的三层分层结构：UI层、业务层、和基础设施层。上层对于下层有直接的依赖关系，导致耦合度过高。在业务层中对于下层的基础设施有强依赖，耦合度高。我们需要对这张图上的每个节点做抽象和整理，来降低对外部依赖的耦合度。


## 抽象数据存储层
第一步常见的操作是将Data Access层做抽象，降低系统对数据库的直接依赖。具体的方法如下：

新建Account实体对象：一个实体（Entity）是拥有ID的域对象，除了拥有数据之外，同时拥有行为。Entity和数据库储存格式无关，在设计中要以该领域的通用严谨语言（Ubiquitous Language）为依据。
新建对象储存接口类AccountRepository：Repository只负责Entity对象的存储和读取，而Repository的实现类完成数据库存储的细节。通过加入Repository接口，底层的数据库连接可以通过不同的实现类而替换。


具体的简单代码实现如下：

Account实体类：
```
@Data
public class Account {
    private AccountId id;
    private AccountNumber accountNumber;
    private UserId userId;
    private Money available;
    private Money dailyLimit;

    public void withdraw(Money money) {
        // 转出
    }

    public void deposit(Money money) {
        // 转入
    }
}
```
和AccountRepository及MyBatis实现类：
```
public interface AccountRepository {
    Account find(AccountId id);
    Account find(AccountNumber accountNumber);
    Account find(UserId userId);
    Account save(Account account);
}

public class AccountRepositoryImpl implements AccountRepository {

    @Autowired
    private AccountMapper accountDAO;

    @Autowired
    private AccountBuilder accountBuilder;

    @Override
    public Account find(AccountId id) {
        AccountDO accountDO = accountDAO.selectById(id.getValue());
        return accountBuilder.toAccount(accountDO);
    }

    @Override
    public Account find(AccountNumber accountNumber) {
        AccountDO accountDO = accountDAO.selectByAccountNumber(accountNumber.getValue());
        return accountBuilder.toAccount(accountDO);
    }

    @Override
    public Account find(UserId userId) {
        AccountDO accountDO = accountDAO.selectByUserId(userId.getId());
        return accountBuilder.toAccount(accountDO);
    }

    @Override
    public Account save(Account account) {
        AccountDO accountDO = accountBuilder.fromAccount(account);
        if (accountDO.getId() == null) {
            accountDAO.insert(accountDO);
        } else {
            accountDAO.update(accountDO);
        }
        return accountBuilder.toAccount(accountDO);
    }

}
```

Account实体类和AccountDO数据类的对比如下：

- Data Object数据类：AccountDO是单纯的和数据库表的映射关系，每个字段对应数据库表的一个column，这种对象叫Data Object。DO只有数据，没有行为。AccountDO的作用是对数据库做快速映射，避免直接在代码里写SQL。无论你用的是MyBatis还是Hibernate这种ORM，从数据库来的都应该先直接映射到DO上，但是代码里应该完全避免直接操作 DO。
- Entity实体类：Account 是基于领域逻辑的实体类，它的字段和数据库储存不需要有必然的联系。Entity包含数据，同时也应该包含行为。在 Account 里，字段也不仅仅是String等基础类型，而应该尽可能用上一讲的 Domain Primitive 代替，可以避免大量的校验代码。


DAO 和 Repository 类的对比如下：

- DAO对应的是一个特定的数据库类型的操作，相当于SQL的封装。所有操作的对象都是DO类，所有接口都可以根据数据库实现的不同而改变。比如，insert 和 update 属于数据库专属的操作。
- Repository对应的是Entity对象读取储存的抽象，在接口层面做统一，不关注底层实现。比如，通过 save 保存一个Entity对象，但至于具体是 insert 还是 update 并不关心。Repository的具体实现类通过调用DAO来实现各种操作，通过Builder/Factory对象实现AccountDO 到 Account之间的转化

### Repository和Entity
- 通过Account对象，避免了其他业务逻辑代码和数据库的直接耦合，避免了当数据库字段变化时，大量业务逻辑也跟着变的问题。
- 通过Repository，改变业务代码的思维方式，让业务逻辑不再面向数据库编程，而是面向领域模型编程。
- Account属于一个完整的内存中对象，可以比较容易的做完整的测试覆盖，包含其行为。
- Repository作为一个接口类，可以比较容易的实现Mock或Stub，可以很容易测试。
- AccountRepositoryImpl实现类，由于其职责被单一出来，只需要关注Account到AccountDO的映射关系和Repository方法到DAO方法之间的映射关系，相对于来说更容易测试。

 

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-1ee42f0915a33f6f000707c974792495_1440w.jpeg)

## 抽象第三方服务
类似对于数据库的抽象，所有第三方服务也需要通过抽象解决第三方服务不可控，入参出参强耦合的问题。在这个例子里我们抽象出 ExchangeRateService 的服务，和一个ExchangeRate的Domain Primitive类：

```
public interface ExchangeRateService {
    ExchangeRate getExchangeRate(Currency source, Currency target);
}

public class ExchangeRateServiceImpl implements ExchangeRateService {

    @Autowired
    private YahooForexService yahooForexService;

    @Override
    public ExchangeRate getExchangeRate(Currency source, Currency target) {
        if (source.equals(target)) {
            return new ExchangeRate(BigDecimal.ONE, source, target);
        }
        BigDecimal forex = yahooForexService.getExchangeRate(source.getValue(), target.getValue());
        return new ExchangeRate(forex, source, target);
    }
}
```

### 防腐层（ACL）
这种常见的设计模式叫做Anti-Corruption Layer（防腐层或ACL）。很多时候我们的系统会去依赖其他的系统，而被依赖的系统可能包含不合理的数据结构、API、协议或技术实现，如果对外部系统强依赖，会导致我们的系统被”腐蚀“。这个时候，通过在系统间加入一个防腐层，能够有效的隔离外部依赖和内部逻辑，无论外部如何变更，内部代码可以尽可能的保持不变。

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-952c66bf1d97e78af66873dd50fe124d_1440w.jpeg)

ACL 不仅仅只是多了一层调用，在实际开发中ACL能够提供更多强大的功能：

- 适配器：很多时候外部依赖的数据、接口和协议并不符合内部规范，通过适配器模式，可以将数据转化逻辑封装到ACL内部，降低对业务代码的侵入。在这个案例里，我们通过封装了ExchangeRate和Currency对象，转化了对方的入参和出参，让入参出参更符合我们的标准。
- 缓存：对于频繁调用且数据变更不频繁的外部依赖，通过在ACL里嵌入缓存逻辑，能够有效的降低对于外部依赖的请求压力。同时，很多时候缓存逻辑是写在业务代码里的，通过将缓存逻辑嵌入ACL，能够降低业务代码的复杂度。
- 兜底：如果外部依赖的稳定性较差，一个能够有效提升我们系统稳定性的策略是通过ACL起到兜底的作用，比如当外部依赖出问题后，返回最近一次成功的缓存或业务兜底数据。这种兜底逻辑一般都比较复杂，如果散落在核心业务代码中会很难维护，通过集中在ACL中，更加容易被测试和修改。
- 易于测试：类似于之前的Repository，ACL的接口类能够很容易的实现Mock或Stub，以便于单元测试。
- 功能开关：有些时候我们希望能在某些场景下开放或关闭某个接口的功能，或者让某个接口返回一个特定的值，我们可以在ACL配置功能开关来实现，而不会对真实业务代码造成影响。同时，使用功能开关也能让我们容易的实现Monkey测试，而不需要真正物理性的关闭外部依赖。

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-dbec337d0e361426cc58864e67756adb_1440w.jpeg)

## 抽象中间件
类似于2.2的第三方服务的抽象，对各种中间件的抽象的目的是让业务代码不再依赖中间件的实现逻辑。因为中间件通常需要有通用型，中间件的接口通常是String或Byte[] 类型的，导致序列化/反序列化逻辑通常和业务逻辑混杂在一起，造成胶水代码。通过中间件的ACL抽象，减少重复胶水代码。

在这个案例里，我们通过封装一个抽象的AuditMessageProducer和AuditMessage DP对象，实现对底层kafka实现的隔离：

```
@Value
@AllArgsConstructor
public class AuditMessage {

    private UserId userId;
    private AccountNumber source;
    private AccountNumber target;
    private Money money;
    private Date date;

    public String serialize() {
        return userId + "," + source + "," + target + "," + money + "," + date;   
    }

    public static AuditMessage deserialize(String value) {
        // todo
        return null;
    }
}

public interface AuditMessageProducer {
    SendResult send(AuditMessage message);
}

public class AuditMessageProducerImpl implements AuditMessageProducer {

    private static final String TOPIC_AUDIT_LOG = "TOPIC_AUDIT_LOG";

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Override
    public SendResult send(AuditMessage message) {
        String messageBody = message.serialize();
        kafkaTemplate.send(TOPIC_AUDIT_LOG, messageBody);
        return SendResult.success();
    }
}
```
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-24099ac4f31bf9472bcf8b19ffaf48ab_1440w.jpeg)

## 封装业务逻辑
### 用Domain Primitive封装跟实体无关的无状态计算逻辑
在这个案例里使用ExchangeRate来封装汇率计算逻辑：
```
BigDecimal exchangeRate = BigDecimal.ONE;
if (sourceAccountDO.getCurrency().equals(targetCurrency)) {
    exchangeRate = yahooForex.getExchangeRate(sourceAccountDO.getCurrency(), targetCurrency);
}
BigDecimal sourceAmount = targetAmount.divide(exchangeRate, RoundingMode.DOWN);
```
变为：
```
ExchangeRate exchangeRate = exchangeRateService.getExchangeRate(sourceAccount.getCurrency(), targetMoney.getCurrency());
Money sourceMoney = exchangeRate.exchangeTo(targetMoney);
```

### 用Entity封装单对象的有状态的行为，包括业务校验
用Account实体类封装所有Account的行为，包括业务校验如下：

```
@Data
public class Account {

    private AccountId id;
    private AccountNumber accountNumber;
    private UserId userId;
    private Money available;
    private Money dailyLimit;

    public Currency getCurrency() {
        return this.available.getCurrency();
    }

    // 转入
    public void deposit(Money money) {
        if (!this.getCurrency().equals(money.getCurrency())) {
            throw new InvalidCurrencyException();
        }
        this.available = this.available.add(money);
    }

    // 转出
    public void withdraw(Money money) {
        if (this.available.compareTo(money) < 0) {
            throw new InsufficientFundsException();
        }
        if (this.dailyLimit.compareTo(money) < 0) {
            throw new DailyLimitExceededException();
        }
        this.available = this.available.subtract(money);
    }
}
```
原有的业务代码则可以简化为：
```
sourceAccount.deposit(sourceMoney);
targetAccount.withdraw(targetMoney);
```

### 用Domain Service封装多对象逻辑
在这个案例里，我们发现这两个账号的转出和转入实际上是一体的，也就是说这种行为应该被封装到一个对象中去。特别是考虑到未来这个逻辑可能会产生变化：比如增加一个扣手续费的逻辑。这个时候在原有的TransferService中做并不合适，在任何一个Entity或者Domain Primitive里也不合适，需要有一个新的类去包含跨域对象的行为。这种对象叫做Domain Service。

我们创建一个AccountTransferService的类：
```
public interface AccountTransferService {
    void transfer(Account sourceAccount, Account targetAccount, Money targetMoney, ExchangeRate exchangeRate);
}

public class AccountTransferServiceImpl implements AccountTransferService {
    private ExchangeRateService exchangeRateService;

    @Override
    public void transfer(Account sourceAccount, Account targetAccount, Money targetMoney, ExchangeRate exchangeRate) {
        Money sourceMoney = exchangeRate.exchangeTo(targetMoney);
        sourceAccount.deposit(sourceMoney);
        targetAccount.withdraw(targetMoney);
    }
}
```
而原始代码则简化为一行：
```
accountTransferService.transfer(sourceAccount, targetAccount, targetMoney, exchangeRate);
```
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-14d91a1f537b39307cf72a437ff3c60f_1440w.jpeg)

## 重构后结果分析
```
public class TransferServiceImplNew implements TransferService {

    private AccountRepository accountRepository;
    private AuditMessageProducer auditMessageProducer;
    private ExchangeRateService exchangeRateService;
    private AccountTransferService accountTransferService;

    @Override
    public Result<Boolean> transfer(Long sourceUserId, String targetAccountNumber, BigDecimal targetAmount, String targetCurrency) {
        // 参数校验
        Money targetMoney = new Money(targetAmount, new Currency(targetCurrency));

        // 读数据
        Account sourceAccount = accountRepository.find(new UserId(sourceUserId));
        Account targetAccount = accountRepository.find(new AccountNumber(targetAccountNumber));
        ExchangeRate exchangeRate = exchangeRateService.getExchangeRate(sourceAccount.getCurrency(), targetMoney.getCurrency());

        // 业务逻辑
        accountTransferService.transfer(sourceAccount, targetAccount, targetMoney, exchangeRate);

        // 保存数据
        accountRepository.save(sourceAccount);
        accountRepository.save(targetAccount);

        // 发送审计消息
        AuditMessage message = new AuditMessage(sourceAccount, targetAccount, targetMoney);
        auditMessageProducer.send(message);

        return Result.success(true);
    }
}
```
可以看出来，经过重构后的代码有以下几个特征：

- 业务逻辑清晰，数据存储和业务逻辑完全分隔。
- Entity、Domain Primitive、Domain Service都是独立的对象，没有任何外部依赖，但是却包含了所有核心业务逻辑，可以单独完整测试。
- 原有的TransferService不再包括任何计算逻辑，仅仅作为组件编排，所有逻辑均delegate到其他组件。这种仅包含Orchestration（编排）的服务叫做Application Service（应用服务）。


我们可以根据新的结构重新画一张图：

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-44ebd98e1697c75e119aa1d94a8f7768_1440w.jpeg)

然后通过重新编排后该图变为：

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-5182bc6ecf7fe977b5968d9181a414aa_1440w.jpeg)

我们可以发现，通过对外部依赖的抽象和内部逻辑的封装重构，应用整体的依赖关系变了：

- 最底层不再是数据库，而是Entity、Domain Primitive和Domain Service。这些对象不依赖任何外部服务和框架，而是纯内存中的数据和操作。这些对象我们打包为Domain Layer（领域层）。领域层没有任何外部依赖关系。
- 再其次的是负责组件编排的Application Service，但是这些服务仅仅依赖了一些抽象出来的ACL类和Repository类，而其具体实现类是通过依赖注入注进来的。Application Service、Repository、ACL等我们统称为Application Layer（应用层）。应用层 依赖 领域层，但不依赖具体实现。
- 最后是ACL，Repository等的具体实现，这些实现通常依赖外部具体的技术实现和框架，所以统称为Infrastructure Layer（基础设施层）。Web框架里的对象如Controller之类的通常也属于基础设施层。
- 

如果今天能够重新写这段代码，考虑到最终的依赖关系，我们可能先写Domain层的业务逻辑，然后再写Application层的组件编排，最后才写每个外部依赖的具体实现。这种架构思路和代码组织结构就叫做Domain-Driven Design（领域驱动设计，或DDD）。所以DDD不是一个特殊的架构设计，而是所有Transction Script代码经过合理重构后一定会抵达的终点。

## DDD的六边形架构

在我们传统的代码里，我们一般都很注重每个外部依赖的实现细节和规范，但是今天我们需要敢于抛弃掉原有的理念，重新审视代码结构。在上面重构的代码里，如果抛弃掉所有Repository、ACL、Producer等的具体实现细节，我们会发现每一个对外部的抽象类其实就是输入或输出，类似于计算机系统中的I/O节点。这个观点在CQRS架构中也同样适用，将所有接口分为Command（输入）和Query（输出）两种。除了I/O之外其他的内部逻辑，就是应用业务的核心逻辑。基于这个基础，Alistair Cockburn在2005年提出了Hexagonal Architecture（六边形架构），又被称之为Ports and Adapters（端口和适配器架构）。

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-11af1ddff05b42025e83e204e71c9a5b_1440w.png)

在这张图中：

- I/O的具体实现在模型的最外层
- 每个I/O的适配器在灰色地带
- 每个Hex的边是一个端口
- Hex的中央是应用的核心领域模型


在Hex中，架构的组织关系第一次变成了一个二维的内外关系，而不是传统一维的上下关系。同时在Hex架构中我们第一次发现UI层、DB层、和各种中间件层实际上是没有本质上区别的，都只是数据的输入和输出，而不是在传统架构中的最上层和最下层。

除了2005年的Hex架构，2008年 Jeffery Palermo的Onion Architecture（洋葱架构）和2017年 Robert Martin的Clean Architecture（干净架构），都是极为类似的思想。除了命名不一样、切入点不一样之外，其他的整体架构都是基于一个二维的内外关系。这也说明了基于DDD的架构最终的形态都是类似的。Herberto Graca有一个很全面的图包含了绝大部分现实中的端口类，值得借鉴。
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-9e83433ede22b4d45e08c067b02e661f_1440w.jpeg)

### 代码组织结构
为了有效的组织代码结构，避免下层代码依赖到上层实现的情况，在Java中我们可以通过POM Module和POM依赖来处理相互的关系。通过Spring/SpringBoot的容器来解决运行时动态注入具体实现的依赖的问题。一个简单的依赖关系图如下：
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/123312kjsalkdkal.jpeg)

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-b3d0dccea152b7e846dc60ca3d615093_1440w.jpeg)

#### Types 模块
Types模块是保存可以对外暴露的Domain Primitives的地方。Domain Primitives因为是无状态的逻辑，可以对外暴露，所以经常被包含在对外的API接口中，需要单独成为模块。Types模块不依赖任何类库，纯 POJO 。


![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-59a53da7e651770e2e1d96bafdf6a503_1440w.jpeg)

#### Domain 模块
Domain 模块是核心业务逻辑的集中地，包含有状态的Entity、领域服务Domain Service、以及各种外部依赖的接口类（如Repository、ACL、中间件等。Domain模块仅依赖Types模块，也是纯 POJO 。

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-1c36105863c3f5187260a02548dd813b_1440w.jpeg)


#### Application模块
Application模块主要包含Application Service和一些相关的类。Application模块依赖Domain模块。还是不依赖任何框架，纯POJO。
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-b9a185f16cd6203796b582e7c43479d3_1440w.jpeg)


#### Infrastructure模块
Infrastructure模块包含了Persistence、Messaging、External等模块。比如：Persistence模块包含数据库DAO的实现，包含Data Object、ORM Mapper、Entity到DO的转化类等。Persistence模块要依赖具体的ORM类库，比如MyBatis。如果需要用Spring-Mybatis提供的注解方案，则需要依赖Spring。

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-31fb63320e200bd58ec087ba4cab195d_1440w.jpeg)

####  Web模块
Web模块包含Controller等相关代码。如果用SpringMVC则需要依赖Spring。
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/v2-71623c8f80edb4f7a6a0bceb8ebe85f5_1440w.jpeg)

#### Start模块
Start模块是SpringBoot的启动类。

## 测试
- Types，Domain模块都属于无外部依赖的纯POJO，基本上都可以100%的被单元测试覆盖。
- Application模块的代码依赖外部抽象类，需要通过测试框架去Mock所有外部依赖，但仍然可以100%被单元测试。
- Infrastructure的每个模块的代码相对独立，接口数量比较少，相对比较容易写单测。但是由于依赖了外部I/O，速度上不可能很快，但好在模块的变动不会很频繁，属于一劳永逸。
- Web模块有两种测试方法：通过Spring的MockMVC测试，或者通过HttpClient调用接口测试。但是在测试时最好把Controller依赖的服务类都Mock掉。一般来说当你把Controller的逻辑都后置到Application Service中时，Controller的逻辑变得极为简单，很容易100%覆盖。
- Start模块：通常应用的集成测试写在start里。当其他模块的单元测试都能100%覆盖后，集成测试用来验证整体链路的真实性。


## 代码的演进/变化速度
在传统架构中，代码从上到下的变化速度基本上是一致的，改个需求需要从接口、到业务逻辑、到数据库全量变更，而第三方变更可能会导致整个代码的重写。但是在DDD中不同模块的代码的演进速度是不一样的：

- Domain层属于核心业务逻辑，属于经常被修改的地方。比如：原来不需要扣手续费，现在需要了之类的。通过Entity能够解决基于单个对象的逻辑变更，通过Domain Service解决多个对象间的业务逻辑变更。
- Application层属于Use Case（业务用例）。业务用例一般都是描述比较大方向的需求，接口相对稳定，特别是对外的接口一般不会频繁变更。添加业务用例可以通过新增Application Service或者新增接口实现功能的扩展。
- Infrastructure层属于最低频变更的。一般这个层的模块只有在外部依赖变更了之后才会跟着升级，而外部依赖的变更频率一般远低于业务逻辑的变更频率。
所以在DDD架构中，能明显看出越外层的代码越稳定，越内层的代码演进越快，真正体现了领域“驱动”的核心思想。

## 总结

DDD不是一个什么特殊的架构，而是任何传统代码经过合理的重构之后最终一定会抵达的终点。DDD的架构能够有效的解决传统架构中的问题：

- 高可维护性：当外部依赖变更时，内部代码只用变更跟外部对接的模块，其他业务逻辑不变。
- 高可扩展性：做新功能时，绝大部分的代码都能复用，仅需要增加核心业务逻辑即可。
- 高可测试性：每个拆分出来的模块都符合单一性原则，绝大部分不依赖框架，可以快速的单元测试，做到100%覆盖。
- 代码结构清晰：通过POM module可以解决模块间的依赖关系， 所有外接模块都可以单独独立成Jar包被复用。当团队形成规范后，可以快速的定位到相关代码。

# Repository模式

为什么要用 Repository

> 实体模型 vs. 贫血模型

Entity（实体）这个词在计算机领域的最初应用可能是来自于Peter Chen在1976年的“The Entity-Relationship Model - Toward a Unified View of Data"（ER模型），用来描述实体之间的关系，而ER模型后来逐渐的演变成为一个数据模型，在关系型数据库中代表了数据的储存方式。而2006年的JPA标准，通过@Entity等注解，以及Hibernate等ORM框架的实现，让很多Java开发对Entity的理解停留在了数据映射层面，忽略了Entity实体的本身行为，造成今天很多的模型仅包含了实体的数据和属性，而所有的业务逻辑都被分散在多个服务、Controller、Utils工具类中，这个就是Martin Fowler所说的的Anemic Domain Model（贫血领域模型）。如何知道你的模型是贫血的呢？可以看一下你代码中是否有以下的几个特征：

- 有大量的XxxDO对象：这里DO虽然有时候代表了Domain Object，但实际上仅仅是数据库表结构的映射，里面没有包含（或包含了很少的）业务逻辑；
- 服务和Controller里有大量的业务逻辑：比如校验逻辑、计算逻辑、格式转化逻辑、对象关系逻辑、数据存储逻辑等；
- 大量的Utils工具类等。


而贫血模型的缺陷是非常明显的：


- 无法保护模型对象的完整性和一致性：因为对象的所有属性都是公开的，只能由调用方来维护模型的一致性，而这个是没有保障的；之前曾经出现的案例就是调用方没有能维护模型数据的一致性，导致脏数据使用时出现bug，这一类的 bug还特别隐蔽，很难排查到。
- 对象操作的可发现性极差：单纯从对象的属性上很难看出来都有哪些业务逻辑，什么时候可以被调用，以及可以赋值的边界是什么；比如说，Long类型的值是否可以是0或者负数？
- 代码逻辑重复：比如校验逻辑、计算逻辑，都很容易出现在多个服务、多个代码块里，提升维护成本和bug出现的概率；一类常见的bug就是当贫血模型变更后，校验逻辑由于出现在多个地方，没有能跟着变，导致校验失败或失效。
- 代码的健壮性差：比如一个数据模型的变化可能导致从上到下的所有代码的变更。
- 强依赖底层实现：业务代码里强依赖了底层数据库、网络/中间件协议、第三方服务等，造成核心逻辑代码的僵化且维护成本高。


虽然贫血模型有很大的缺陷，但是在我们日常的代码中，我见过的99%的代码都是基于贫血模型，为什么呢？我总结了以下几点：

- 数据库思维：从有了数据库的那一天起，开发人员的思考方式就逐渐从“写业务逻辑“转变为了”写数据库逻辑”，也就是我们经常说的在写CRUD代码。
- 贫血模型“简单”：贫血模型的优势在于“简单”，仅仅是对数据库表的字段映射，所以可以从前到后用统一格式串通。这里简单打了引号，是因为它只是表面上的简单，实际上当未来有模型变更时，你会发现其实并不简单，每次变更都是非常复杂的事情
- 脚本思维：很多常见的代码都属于“脚本”或“胶水代码”，也就是流程式代码。脚本代码的好处就是比较容易理解，但长久来看缺乏健壮性，维护成本会越来越高。


但是可能最核心的原因在于，实际上我们在日常开发中，混淆了两个概念：

- 数据模型（Data Model）：指业务数据该如何持久化，以及数据之间的关系，也就是传统的ER模型；
- 业务模型/领域模型（Domain Model）：指业务逻辑中，相关联的数据该如何联动。



所以，解决这个问题的根本方案，就是要在代码里严格区分Data Model和Domain Model，具体的规范会在后文详细描述。在真实代码结构中，Data Model和 Domain Model实际上会分别在不同的层里，Data Model只存在于数据层，而Domain Model在领域层，而链接了这两层的关键对象，就是Repository。


> Repository的价值

在传统的数据库驱动开发中，我们会对数据库操作做一个封装，一般叫做Data Access Object（DAO）。DAO的核心价值是封装了拼接SQL、维护数据库连接、事务等琐碎的底层逻辑，让业务开发可以专注于写代码。但是在本质上，DAO的操作还是数据库操作，DAO的某个方法还是在直接操作数据库和数据模型，只是少写了部分代码。在Uncle Bob的《代码整洁之道》一书里，作者用了一个非常形象的描述：

- 硬件（Hardware）：指创造了之后不可（或者很难）变更的东西。数据库对于开发来说，就属于”硬件“，数据库选型后基本上后面不会再变，比如：用了MySQL就很难再改为MongoDB，改造成本过高。
- 软件（Software）：指创造了之后可以随时修改的东西。对于开发来说，业务代码应该追求做”软件“，因为业务流程、规则在不停的变化，我们的代码也应该能随时变化。
- 固件（Firmware）：即那些强烈依赖了硬件的软件。我们常见的是路由器里的固件或安卓的固件等等。固件的特点是对硬件做了抽象，但仅能适配某款硬件，不能通用。所以今天不存在所谓的通用安卓固件，而是每个手机都需要有自己的固件。
- 

从上面的描述我们能看出来，数据库在本质上属于”硬件“，DAO 在本质上属于”固件“，而我们自己的代码希望是属于”软件“。但是，固件有个非常不好的特性，那就是会传播，也就是说当一个软件强依赖了固件时，由于固件的限制，会导致软件也变得难以变更，最终让软件变得跟固件一样难以变更。

## 模型对象代码规范
这边要解决这个问题，首先要指定编码规约

> 对象类型

在讲Repository规范之前，我们需要先讲清楚3种模型的区别，Entity、Data Object (DO)和Data Transfer Object (DTO)：

- Data Object （DO、数据对象）：实际上是我们在日常工作中最常见的数据模型。但是在DDD的规范里，DO应该仅仅作为数据库物理表格的映射，不能参与到业务逻辑中。为了简单明了，DO的字段类型和名称应该和数据库物理表格的字段类型和名称一一对应，这样我们不需要去跑到数据库上去查一个字段的类型和名称。（当然，实际上也没必要一摸一样，只要你在Mapper那一层做到字段映射）
- Entity（实体对象）：实体对象是我们正常业务应该用的业务模型，它的字段和方法应该和业务语言保持一致，和持久化方式无关。也就是说，Entity和DO很可能有着完全不一样的字段命名和字段类型，甚至嵌套关系。Entity的生命周期应该仅存在于内存中，不需要可序列化和可持久化。
- DTO（传输对象）：主要作为Application层的入参和出参，比如CQRS里的Command、Query、Event，以及Request、Response等都属于DTO的范畴。DTO的价值在于适配不同的业务场景的入参和出参，避免让业务对象变成一个万能大对象。




## 模型所在模块和转化器

于现在从一个对象变为3+个对象，对象间需要通过转化器（Converter/Mapper）来互相转化。而这三种对象在代码中所在的位置也不一样，简单总结如下：
![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/v2-93a7e4f3d3bda78cf8ba33411a5d3e2d_1440w.jpeg)



> 这边DO结尾的文件是跟数据库一一对应的，DTO是跟前端或者其他系统协商的，只有DTO需要序列化，Entity是不固定的，实际的属性设置是根据业务来的。这边我推荐用一个mapper层，所有文件都以Mapper结尾，在Mapper层做转换，这边具体的转换用一个工具很好用，叫MapStruct。

从使用复杂度角度来看，区分了DO、Entity、DTO带来了代码量的膨胀（从1个变成了3+2+N个）。但是在实际复杂业务场景下，通过功能来区分模型带来的价值是功能性的单一和可测试、可预期，最终反而是逻辑复杂性的降低。

我自己的项目因为是简单的业务模型，所以用了通用的贫血模型，规范的项目目录结构如下所示（这边缺少了entity和防腐层的设计）：

![image](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/common/WechatIMG629.png)

# DDD领域层的一些设计规范

上面我主要针对同一个例子对比了OOP、ECS和DDD的3种实现，比较如下：

- 基于继承关系的OOP代码：OOP的代码最好写，也最容易理解，所有的规则代码都写在对象里，但是当领域规则变得越来越复杂时，其结构会限制它的发展。新的规则有可能会导致代码的整体重构。
- 基于组件化的ECS代码：ECS代码有最高的灵活性、可复用性、及性能，但极具弱化了实体类的内聚，所有的业务逻辑都写在了服务里，会导致业务的一致性无法保障，对商业系统会有较大的影响。
- 基于领域对象 + 领域服务的DDD架构：DDD的规则其实最复杂，同时要考虑到实体类的内聚和保证不变性（Invariants），也要考虑跨对象规则代码的归属，甚至要考虑到具体领域服务的调用方式，理解成本比较高。


## 实体类（Entity）


大多数DDD架构的核心都是实体类，实体类包含了一个领域里的状态、以及对状态的直接操作。Entity最重要的设计原则是保证实体的不变性（Invariants），也就是说要确保无论外部怎么操作，一个实体内部的属性都不能出现相互冲突，状态不一致的情况。所以几个设计原则如下：

创建即一致
在贫血模型里，通常见到的代码是一个模型通过手动new出来之后，由调用方一个参数一个参数的赋值，这就很容易产生遗漏，导致实体状态不一致。所以DDD里实体创建的方法有两种：

constructor参数要包含所有必要属性，或者在constructor里有合理的默认值。

> 使用Factory模式来降低调用方复杂度

### 尽量避免public setter
一个最容易导致不一致性的原因是实体暴露了public的setter方法，特别是set单一参数会导致状态不一致的情况。比如，一个订单可能包含订单状态（下单、已支付、已发货、已收货）、支付单、物流单等子实体，如果一个调用方能随意去set订单状态，就有可能导致订单状态和子实体匹配不上，导致业务流程走不通的情况。所以在实体里，需要通过行为方法来修改内部状态。

### 通过聚合根保证主子实体的一致性
在稍微复杂一点的领域里，通常主实体会包含子实体，这时候主实体就需要起到聚合根的作用，即：

- 子实体不能单独存在，只能通过聚合根的方法获取到。任何外部的对象都不能直接保留子实体的引用
- 子实体没有独立的Repository，不可以单独保存和取出，必须要通过聚合根的Repository实例化
- 子实体可以单独修改自身状态，但是多个子实体之间的状态一致性需要聚合根来保障


常见的电商域中聚合的案例如主子订单模型、商品/SKU模型、跨子订单优惠、跨店优惠模型等。很多聚合根和Repository的设计规范在我前面一篇关于Repository的文章中已经详细解释过，可以拿来参考。

### 不可以强依赖其他聚合根实体或领域服务
一个实体的原则是高内聚、低耦合，即一个实体类不能直接在内部直接依赖一个外部的实体或服务。这个原则和绝大多数ORM框架都有比较严重的冲突，所以是一个在开发过程中需要特别注意的。这个原则的必要原因包括：对外部对象的依赖性会直接导致实体无法被单测；以及一个实体无法保证外部实体变更后不会影响本实体的一致性和正确性。



所以，正确的对外部依赖的方法有两种：

- 只保存外部实体的ID：这里我再次强烈建议使用强类型的ID对象，而不是Long型ID。强类型的ID对象不单单能自我包含验证代码，保证ID值的正确性，同时还能确保各种入参不会因为参数顺序变化而出bug。具体可以参考Domain Primitive。
- 针对于“无副作用”的外部依赖，通过方法入参的方式传入。比如上文中的equip(Weapon，EquipmentService）方法。


如果方法对外部依赖有副作用，不能通过方法入参的方式，只能通过Domain Service解决，见下文。

### 任何实体的行为只能直接影响到本实体（和其子实体）
这个原则更多是一个确保代码可读性、可理解的原则，即任何实体的行为不能有“直接”的”副作用“，即直接修改其他的实体类。这么做的好处是代码读下来不会产生意外。



另一个遵守的原因是可以降低未知的变更的风险。在一个系统里一个实体对象的所有变更操作应该都是预期内的，如果一个实体能随意被外部直接修改的话，会增加代码bug的风险。

## 领域服务（Domain Service）
在上文讲到，领域服务其实也分很多种，在这里根据上文总结出来三种常见的：

### 单对象策略型
这种领域对象主要面向的是单个实体对象的变更，但涉及到多个领域对象或外部依赖的一些规则。在上文中，EquipmentService即为此类：

- 变更的对象是Player的参数
- 读取的是Player和Weapon的数据，可能还包括从外部读取一些数据

在这种类型下，实体应该通过方法入参的方式传入这种领域服务，然后通过Double Dispatch来反转调用领域服务的方法。


### 跨对象事务型
当一个行为会直接修改多个实体时，不能再通过单一实体的方法作处理，而必须直接使用领域服务的方法来做操作。在这里，领域服务更多的起到了跨对象事务的作用，确保多个实体的变更之间是有一致性的。

### 通用组件型
这种类型的领域服务更像ECS里的System，提供了组件化的行为，但本身又不直接绑死在一种实体类上。

## 策略对象（Domain Policy）

Policy或者Strategy设计模式是一个通用的设计模式，但是在DDD架构中会经常出现，其核心就是封装领域规则。

一个Policy是一个无状态的单例对象，通常需要至少2个方法：canApply 和 一个业务方法。其中，canApply方法用来判断一个Policy是否适用于当前的上下文，如果适用则调用方会去触发业务方法。通常，为了降低一个Policy的可测试性和复杂度，Policy不应该直接操作对象，而是通过返回计算后的值，在Domain Service里对对象进行操作。


除了本文里静态注入多个Policy以及手动排优先级之外，在日常开发中经常能见到通过Java的SPI机制或类SPI机制注册Policy，以及通过不同的Priority方案对Policy进行排序，在这里就不作太多的展开了。

## 领域事件介绍
领域事件是一个在领域里发生了某些事后，希望领域里其他对象能够感知到的通知机制。在上面的案例里，代码之所以会越来越复杂，其根本的原因是反应代码（比如升级）直接和上面的事件触发条件（比如收到经验）直接耦合，而且这种耦合性是隐性的。领域事件的好处就是将这种隐性的副作用“显性化”，通过一个显性的事件，将事件触发和事件处理解耦，最终起到代码更清晰、扩展性更好的目的。

所以，领域事件是在DDD里，比较推荐使用的跨实体“副作用”传播机制。

### 领域事件实现
和消息队列中间件不同的是，领域事件通常是立即执行的、在同一个进程内、可能是同步或异步。我们可以通过一个EventBus来实现进程内的通知机制，简单实现如下：

```
// 实现者：瑜进 2019/11/28
public class EventBus {

    // 注册器
    @Getter
    private final EventRegistry invokerRegistry = new EventRegistry(this);

    // 事件分发器
    private final EventDispatcher dispatcher = new EventDispatcher(ExecutorFactory.getDirectExecutor());

    // 异步事件分发器
    private final EventDispatcher asyncDispatcher = new EventDispatcher(ExecutorFactory.getThreadPoolExecutor());

    // 事件分发
    public boolean dispatch(Event event) {
        return dispatch(event, dispatcher);
    }

    // 异步事件分发
    public boolean dispatchAsync(Event event) {
        return dispatch(event, asyncDispatcher);
    }

    // 内部事件分发
    private boolean dispatch(Event event, EventDispatcher dispatcher) {
        checkEvent(event);
        // 1.获取事件数组
        Set<Invoker> invokers = invokerRegistry.getInvokers(event);
        // 2.一个事件可以被监听N次，不关心调用结果
        dispatcher.dispatch(event, invokers);
        return true;
    }

    // 事件总线注册
    public void register(Object listener) {
        if (listener == null) {
            throw new IllegalArgumentException("listener can not be null!");
        }
        invokerRegistry.register(listener);
    }

    private void checkEvent(Event event) {
        if (event == null) {
            throw new IllegalArgumentException("event");
        }
        if (!(event instanceof Event)) {
            throw new IllegalArgumentException("Event type must by " + Event.class);
        }
    }
}
```

调用方式：
```
public class LevelUpEvent implements Event {
    private Player player;
}

public class LevelUpHandler {
    public void handle(Player player);
}

public class Player {
    public void receiveExp(int value) {
        this.exp += value;
        if (this.exp >= 100) {
            LevelUpEvent event = new LevelUpEvent(this);
            EventBus.dispatch(event);
            this.exp = 0;
        }
    }
}
@Test
public void test() {
    EventBus.register(new LevelUpHandler());
    player.setLevel(1);
    player.receiveExp(100);
    assertThat(player.getLevel()).equals(2);
}
```

## 目前领域事件的缺陷和展望
从上面代码可以看出来，领域事件的很好的实施依赖EventBus、Dispatcher、Invoker这些属于框架级别的支持。同时另一个问题是因为Entity不能直接依赖外部对象，所以EventBus目前只能是一个全局的Singleton，而大家都应该知道全局Singleton对象很难被单测。这就容易导致Entity对象无法被很容易的被完整单测覆盖全。

另一种解法是侵入Entity，对每个Entity增加一个List:
```
public class Player {
  List<Event> events;

  public void receiveExp(int value) {
        this.exp += value;
        if (this.exp >= 100) {
            LevelUpEvent event = new LevelUpEvent(this);
            events.add(event); // 把event加进去
            this.exp = 0;
        }
    }
}

@Test
public void test() {
    EventBus.register(new LevelUpHandler());
    player.setLevel(1);
    player.receiveExp(100);

    for(Event event: player.getEvents()) { // 在这里显性的dispatch事件
        EventBus.dispatch(event);
    }

    assertThat(player.getLevel()).equals(2);
}
```
但是能看出来这种解法不但会侵入实体本身，同时也需要比较啰嗦的显性在调用方dispatch事件，也不是一个好的解决方案。

也许未来会有一个框架能让我们既不依赖全局Singleton，也不需要显性去处理事件，但目前的方案基本都有或多或少的缺陷，大家在使用中可以注意。

# 总结
在真实的业务逻辑里，我们的领域模型或多或少的都有一定的“特殊性”，如果100%的要符合DDD规范可能会比较累，所以最主要的是梳理一个对象行为的影响面，然后作出设计决策，即：

- 是仅影响单一对象还是多个对象，
- 规则未来的拓展性、灵活性，
- 性能要求，
- 副作用的处理，等等


当然，很多时候一个好的设计是多种因素的取舍，需要大家有一定的积累，真正理解每个架构背后的逻辑和优缺点。一个好的架构师不是有一个正确答案，而是能从多个方案中选出一个最平衡的方案。






参考文章：

- [阿里技术专家详解 DDD 系列 第一讲- Domain Primitive](https://zhuanlan.zhihu.com/p/340911587)
- [阿里技术专家详解DDD系列 第二讲 - 应用架构](https://zhuanlan.zhihu.com/p/343388831)
- [阿里技术专家详解DDD系列 第三讲 - Repository模式](https://zhuanlan.zhihu.com/p/348706530)
- [阿里技术专家详解DDD系列 第四讲 - 领域层设计规范](https://zhuanlan.zhihu.com/p/356518017)