# 使用 Red Hat Decision Manager 7 检测信用卡欺诈

> 原文：<https://developers.redhat.com/blog/2018/07/26/detecting-credit-card-fraud-with-red-hat-decision-manager-7>

[Red Hat Decision Manager](https://developers.redhat.com/products/red-hat-decision-manager/overview/) 提供了大量的决策管理功能。从新的决策模型和符号(DMN)1.1 版中的决策表功能(它实现了 DMN 规范的完整感觉遵从级别 3)到预测模型标记语言(PMML)。

另一个强大的功能是复杂事件处理(CEP)引擎。该引擎提供了检测、关联、抽象、聚合或合成事件并对其做出反应的能力。换句话说，该技术提供了从简单事件推断复杂事件、对感兴趣的事件做出反应并采取行动的技术。CEP 和普通的规则执行的主要区别在于时间的概念。决策管理器中的标准规则执行处理事实和对这些事实的推理，而 CEP 引擎关注事件。事件表示在特定时间点或时间间隔内状态的重大变化。

最近，有人请我演示如何在实时信用卡欺诈检测系统中使用决策管理器 CEP。我遇到的一个需求最终形成了一个有趣的规则实现，它构成了本文的基础。该要求定义如下:

*“当信用卡交易进入系统时，从数据存储中获取该交易的上下文，其中上下文被定义为同一信用卡的{x}次先前交易。如果在当前交易的最后 15 分钟内，同一张卡有三笔或更多的额外交易，并且其中至少两笔交易的时间间隔在 10 秒之内，则发出“潜在欺诈”警报。”*

当我们更详细地查看需求时，我们可以发现一些有趣的基于时间的约束。让我们逐一分析，逐步建立我们的 CEP 规则。让我们假设从数据存储中获取当前事务的上下文(前{x}个事务)的逻辑已经给出，并且事务已经被插入到 CEP 引擎中。

## **约束条件**

1)过去 15 分钟内是否有四笔或更多交易？

1-a)这个需求有一个窗口的概念:更准确地说，是一个基于时间的 15 分钟的窗口，它将捕获在这个时间段内发生的所有信用卡交易。在决策管理器中，这可以使用带有*时间窗口*构造的`accumulate`构造来实现。`accumulate`构造定义如下:

```
accumulate( <source pattern>; <functions> [;<constraints>] )
```

让我们从在我们的时间窗口中累积所有信用卡交易的源模式开始:

```
accumulate ($cct:CreditCardTransaction() over window:time(15m) from entry-point Transactions
```

这表明我们希望从入口点`Transactions`开始累积在过去 15 分钟内发生的所有`CreditCardTransaction`事件。

1-b)现在我们已经定义了累积过去 15 分钟内发生的所有事务的逻辑，我们需要定义所谓的*累积函数*来处理这些数据。如果我们查看需求，我们可以看到我们需要实现一个约束来确定在过去的 15 分钟内是否有 4 个或更多的事务。因此，我们可以使用`count` accumulate 函数来计算我们的时间窗口中的事务数量:

```
accumulate ($cct:CreditCardTransaction() over window:time(15m) from entry-point Transactions;
              $nrOfTransactions: count($cct)
```

1-c)现在我们已经计算了事务的数量，我们可以实现约束来检查我们在这个窗口中是否有四个或更多的事务:

```
accumulate ($cct:CreditCardTransaction() over window:time(15m) from entry-point Transactions;
              $nrOfTransactions: count($cct);
              $nrOfTransactions >= 4)
```

2)该规则的第二个要求是，在四个或更多交易的集合中，至少有两个交易彼此发生在 10 秒内。

2-a)为了满足这个需求，我们需要在我们的窗口中访问`CreditCardTransaction`事件。我们通过使用名为`collectList`的第二个累加函数来实现这一点。

```
accumulate ($cct: CreditCardTransaction() over window:time (15m) from entry-point Transactions;
              $nrOfTransactions : count($cct),
              $list: collectList($cct);
              $nrOfTransactions >= 4)
```

2-b)现在我们可以访问事件的集合(列表),我们可以开始比较它们了。我们需要检查两个`CreditCardTransaction`事件是否在 10 秒内相继发生。为此，我们使用时态操作符`after`来分析两个事件之间的时态距离。这两个约束如下所示:

```
$c1: CreditCardTransaction() from $list
$c2: CreditCardTransaction(this != $c1, this after[0s, 10s] $c1) from $list
```

请注意，第二个约束定义了`CreditCardTransaction`与第一个约束中选择的`CreditCardTransaction`不同，并且此事件发生在另一个事件的`after`10 秒内。

现在，我们可以将约束条件合并到规则的完整左侧(LHS ):

```
accumulate ($cct: CreditCardTransaction() over window:time (15m) from entry-point Transactions;
              $nrOfTransactions : count($cct),
              $list: collectList($cct);
              $nrOfTransactions >= 4)
$c1: CreditCardTransaction() from $list
$c2: CreditCardTransaction(this != $c1, this after[0s, 10s] $c1) from $list
```

## **动作**

实现了约束之后，我们现在可以定义规则的右侧(RHS ),或者说动作。在这个例子中，我们将在规则引擎中引入一个新的事实。这是一个我们称之为*推论*的概念。这使我们能够定义额外的规则，这些规则定义对新推断的信息/事实/事件的约束和动作，从而创建更复杂的规则库。我们的最后一条规则是这样的:

```
rule "CC-Transactions last 15 minutes"
when
    accumulate ($cct: CreditCardTransaction() over window:time (15m) from entry-point Transactions;
                 $nrOfTransactions : count($cct),
                 $list: collectList($cct);
                 $nrOfTransactions >= 4)
    $c1: CreditCardTransaction() from $list
    $c2: CreditCardTransaction(this != $c1, this after[0s, 10s] $c1) from $list
then
    System.out.println("\nFound 4 or more cc transactions in last 15 minutes of current transaction");
    System.out.println("And within that collection, there are 2 transactions within 10 seconds of each other.\n");
    PotentialFraudFact potentialFraud = new PotentialFraudFact();
    potentialFraud.setTransactions(new java.util.ArrayList());
    potentialFraud.setCreditCardNumber($c1.getCreditCardNumber());
    potentialFraud.getTransactions().add($c1);
    potentialFraud.getTransactions().add($c2);
    insert(potentialFraud);
end
```

请注意，`System.out.println`语句仅用于演示目的；不建议在产生式规则中使用这种语句。还要注意，在我们的 RHS 结束时，我们将新的`PotentialFraudFact`放入规则引擎。

## **推论**

将推断的`PotentialFraudFact`插入到 CEP 会话中使我们能够编写以下规则来反映这一事实:

```
rule "Found potential fraud"
when
    exists PotentialFraudFact()
then
    System.out.println("\n!!!! Found a potential fraud!!!!\n");
end
```

此规则定义每当一个或多个规则检测到潜在的欺诈时发出警报。同样，`System.out.println`语句仅用于演示目的。在真实的规则库中，这种规则的动作可以是在 Red Hat Process Automation Manager 案例管理引擎中启动欺诈调查案例，该案例管理引擎:

*   冻结信用卡
*   通知用户
*   并将潜在的欺诈通知给银行的代理人

## **演示项目**

展示本文中规则的演示项目可以在这里找到:[https://github . com/Duncan Doyle/drools-credit-card-fraud-detection-demo](https://github.com/DuncanDoyle/drools-credit-card-fraud-detection-demo)。

这是一个简单的项目，其中信用卡交易是在一个 [CSV 文件](https://github.com/DuncanDoyle/drools-credit-card-fraud-detection-demo/blob/master/src/main/resources/ccTransactions.csv)中定义和加载的。要运行这个项目，只需从 IDE (Eclipse、IntelliJ 等)中运行`Main`类。)或使用以下 Maven 命令:

`mvn clean compile exec:java`

对潜在欺诈的检测将显示在日志中。

## **结论**

[Red Hat Decision Manager 7](https://www.redhat.com/en/technologies/jboss-middleware/decision-manager) 的复杂事件处理引擎允许用户在简洁而集中的规则中对数据流实施复杂的、基于时间的要求，这些规则使用高级构造，如`accumulate`构造、时间窗口构造和时态运算符。我们看到，通过使用像*推理*这样的技术，我们可以从简单的事件中推断出新的、复杂的信息，从而允许我们编写更高级和更大的规则库，这些规则库仍然易于软件工程师和领域专家理解。

当与其他平台和技术结合使用时，Decision Manager 7 CEP 为更先进的实时信用卡欺诈检测系统提供了一个良好的基础，例如:

*   Red Hat Data Grid/Infinispan 用于缓存信用卡的上下文，提供对数据的快速内存访问
*   Vert.x 实现来自不同来源的大量客户端的信用卡事件的异步反应执行
*   红帽 AMQ/卡夫卡，用于事务事件的摄取和流处理

我们将在以后的文章中探索这些架构。

![Duncan Doyle](img/31cf5c8a0bf8e97ac6b3ae99ebeaad6f.png)

### 关于作者:

[Duncan Doyle](http://twitter.com/DuncanDoyle) 是 Red Hat 决策管理器和流程自动化管理器平台的技术营销经理。Duncan 拥有 Red Hat 咨询和服务的背景，曾与大型 Red Hat 客户广泛合作，构建高级、开源、决策管理和业务流程管理解决方案。

他在面向服务的架构、持续集成和交付、规则引擎和 BPM 平台等技术和概念方面有着深厚的背景，是多种 JBoss 中间件技术的主题专家(SME ),包括但不限于 JBoss EAP、HornetQ、Fuse、DataGrid、Decision Manager 和 Process Automation Manager。当他不从事开源解决方案和技术时，他就和他的儿子和女儿一起建造乐高，或者在他的 Fender Stratocaster 上播放一些 90 年代的摇滚音乐。

*Last updated: July 25, 2018*