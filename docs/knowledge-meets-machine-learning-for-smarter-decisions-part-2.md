# 知识与机器学习相结合，实现更明智的决策，第 2 部分

> 原文：<https://developers.redhat.com/blog/2021/01/22/knowledge-meets-machine-learning-for-smarter-decisions-part-2>

[Red Hat Decision Manager](https://developers.redhat.com/products/red-hat-decision-manager/overview) 帮助组织将[人工智能](https://developers.redhat.com/topics/ai-ml)的好处引入到日常运营中。它基于 Drools，这是一个流行的开源项目，以其强大的规则引擎而闻名。

在本文第一部分的[中，我们构建了一个机器学习算法，并将其存储在一个](https://developers.redhat.com/blog/2021/01/14/knowledge-meets-machine-learning-for-smarter-decisions-part-1/)[预测模型标记语言](http://dmg.org/pmml/v4-1/GeneralStructure.html) (PMML)文件中。在第 2 部分中，我们将把机器学习逻辑与使用[决策模型和符号(DMN)模型](https://www.omg.org/dmn/)定义的确定性知识结合起来。DMN 是对象管理小组最近推出的标准。它提供了一种通用的符号来捕获应用程序的决策逻辑，以便业务用户能够理解它。

**注意**:本文中的例子建立在第 1 部分的讨论之上。如果您还没有这样做，请[在继续之前阅读本文的前半部分](https://developers.redhat.com/blog/2021/01/14/knowledge-meets-machine-learning-for-smarter-decisions-part-1/)。

## PMML 的优势

机器学习算法的最终目标是在给定特定输入的情况下预测一个值。正如我在第 1 部分中讨论的，有许多不同的机器学习算法，每一种都有自己的结构、训练选项和逻辑执行。大多数时候，终端用户不需要知道*一个算法如何*获得它的结果；我们只需要知道结果是准确的。

PMML 隐藏了实现细节。它还为我们提供了一个通用语言描述符，我们可以用它来组合用不同工具创建的预测模型。 [sklearn-pmml-model](https://pypi.org/project/sklearn-pmml-model/) 项目集成了 pmml 和`scikit-learn`。

PMML 还将机器学习领域与知识工程领域分开。这种分离使得专家更容易管理每个领域的细节，然后使用公共语言描述符来集成它们。

### JPMML

[JPMML](https://github.com/jpmml/jpmml-transpiler) 是由 [Openscoring.io](https://openscoring.io/) 提供的 PMML 的 [Java](https://developers.redhat.com/topics/enterprise-java) 实现。Drools 和 Red Hat Decision Manager 在执行 DMN 逻辑的同一个进程中使用 JPMML 进行 PMML 执行，使得整个执行过程非常高效。

Drools 和 JPMML 是以不同的开源许可证发布的，JPMML 没有与 Drools 二进制文件和 Red Hat Decision Manager 打包在一起。作为用户，您需要下载 JPMML 库，并将它们放在与您的 Red Hat Decision Manager 实例相关联的 KIE 服务器和商业中心存储库的`lib`文件夹中。

我们的[示例项目的源代码](https://github.com/dmarrazzo/rhdm-dmn-pmml-order)带有一个 Maven 配置，它将所有项目依赖项复制到依赖文件夹中。下面是下载依赖项的命令:

```
mvn dependency:copy-dependencies

```

您需要复制以下库:

```
pmml-evaluator-1.4.9.jar
pmml-agent-1.4.11.jar
pmml-model-1.4.11.jar
pmml-evaluator-extension-1.4.9.jar
kie-dmn-jpmml-7.33.0.Final-redhat-00003.jar

```

最后一项是 Drools 库，它在 DMN 运行时中启用 JPMML。

### 将 PMML 和 DMN 用于机器学习

使用 PMML 的唯一缺点是它更专注于数据科学，而不是机器学习。因此，该规范不包括所有可用的机器学习算法。你仍然可以将 DMN 与机器学习结合使用，但在用户体验方面可能会不太舒服。

事实上，DMN 可以使用外部定义的函数来执行 Java 代码。这种方法允许您利用规范中没有包含的机器学习实现，无论它们是 Java 库还是其他技术。甚至可以调用远程评估，将机器学习执行隔离在单独的微服务中。

## 知识工程遇上机器学习

机器学习算法提供预测。如何处理结果是基于*知识背景*的*决策*。我在第 1 部分中介绍的简单案例研究包括不同产品类型的参考价格表。随着价格的调整，该表会随着时间而变化，这些变化会影响决策结果。

现在，假设我们想要引入一个业务需求，即任何超过 1，500 美元的费用都必须将供应订单提交给经理。该政策将让我们提前知道如何处理更大的费用请求，但是我们应该如何实施它呢？

我们可以训练算法拒绝任何超过 1500 美元的订单，但这将是一个糟糕的选择。当我们有把握的时候，我们不应该依赖预测。换个说法，如果你有明确的政策，就用知识工程，不要用机器学习。

## 示例项目

要在决策中使用 PMML，我们必须将其导入业务中心(也称为决策中心)。图 1 中的图表显示了来自`scikit-learn`的输出如何进入 Red Hat 决策管理器和决策中心。

[![](img/a865d2cae60bd3abe1cd87f82d02d28c.png "img_5fa263e78039f")](/sites/default/files/blog/2020/11/img_5fa263e78039f.png)

Figure 1: Output from Scikit-learn feeds Red Hat Decision Manager.

我们可以将这个项目的 [GitHub 存储库直接导入到决策中心:PMML 文件已经被导入，DMN 文件通过引用包含了它。](https://github.com/dmarrazzo/rhdm-dmn-pmml-order)

**注**:如需快速了解 DMN，参见*[15 分钟学会 DMN](http://learn-dmn-in-15-minutes.com/)*。

### DMN 逻辑

对于这个例子，我们试图保持 DMN 逻辑的最小化，集中在 PMML 集成上，但是有几个特性是值得研究的。首先，考虑图 2 中的决策需求图。

[![A block diagram showing how an order moves from input to approval.](img/be9fb83ff11a42983d47ab3ecd0049b4.png "img_5fa2641fede87")](/sites/default/files/blog/2020/11/img_5fa2641fede87.png)

Figure 2: A decision requirement diagram for automatic approval.

图 3 是对`OrderInfo`数据类型的进一步观察。

[![The OrderInfo data type includes productType, price, category, and urgency.](img/7df783250a6b1e0369bb1810af3e2dd2.png "img_5fa26429a6449")](/sites/default/files/blog/2020/11/img_5fa26429a6449.png)

Figure 3: The structure of the OrderInfo data type.

请注意以下事项:

*   输入数据类别是产品类型、价格、类别和紧急程度。
*   计算出目标价格，并与其他数据一起用于预测。
*   预测触发机器学习调用(ML 调用)。边角被剪短的方框是业务知识模型，代表机器学习算法的执行。
*   最后，自动审批基于预测和附加逻辑。

图 4 中显示的目标价格决策用一个简单的决策表捕获了公司对资产参考价格的策略。

[![The decision table captures the product type and target price for each request.](img/0e09c64a890a22d4667b5d53651f1c3c.png "img_5fa2643375095")](/sites/default/files/blog/2020/11/img_5fa2643375095.png)

Figure 4: The decision table for Target Price.

如图 5 所示，预测决策节点调用机器学习执行(ML 调用)。这个节点可能看起来很复杂。实际上，它将决策的类别和紧迫性转化为数字。当概率超过阈值 0.5 时，机器学习算法返回对*真* ( `probability(true)`)的预测。

[![The Prediction node translates the category and urgency of a decision to numbers.](img/009d3a98cb02eaa01634cb9e343da80b.png "img_5fa264471075d")](/sites/default/files/blog/2020/11/img_5fa264471075d.png)

Figure 5: The implementation of the Prediction decision node.

### 商业知识模型

该项目的业务知识模型很简单，如图 6 所示。

[![When a user chooses the PMML document and model from a drop-down list, PMML introspection automatically infers the input parameters.](img/6bcdce99973f8bc61c01acb7a5922bcf.png "img_5fa2645597879")](/sites/default/files/blog/2020/11/img_5fa2645597879.png)

Figure 6: The business knowledge model calls the machine learning model.

用户从下拉列表中选择 PMML 文档和模型。PMML 自检自动推断输入参数。

### 调用机器学习算法

从决策专家的角度来看，调用机器学习算法很简单:信息契约由 PMML 文件定义并自动导入。如果决策专家需要理解规则的语义(例如,“低”紧急性转化为 0 ),他们可以与数据科学家交流。

对于一个稍微不太明显的规则，考虑如何在 DMN 映射模型结果。我们可以在 PMML 的档案中找到这些线:

```
       <Output>

           <OutputField name="probability(false)" optype="continuous" dataType="double" feature="probability" value="false"/>

           <OutputField name="probability(true)" optype="continuous" dataType="double" feature="probability" value="true"/>

       </Output>

```

它们在以下[足够友好的表达语言](https://docs.camunda.org/manual/7.14/reference/dmn/feel/) (FEEL)上下文中被翻译:

```
{ 

  “probability(true)” : *number*,

  “probability(false)”: *number*

}

```

顶层节点用于最终决定是否自动批准订单。请记住，在第 1 部分中，该决策包括一个简单的公司政策:*当费用低于 1，500 美元*时，可以进行自动批准。下面是如何用一个有感觉表达式来实现该策略:

```
if order info.price < 1500 then 

  Prediction

else

  false

```

图 7 显示了高层次的决策生命周期。请注意，设计阶段是在 Python 和决策中心之间划分的。运行时是 KIE 服务器(也称为决策中心)。

[![A block diagram showing the toolchain progression from Scikit-learn to Decision Central, to Kie Server.](img/ed9f7fca879130e7441b3fdad62bd328.png "img_5fa2647888bf2")](/sites/default/files/blog/2020/11/img_5fa2647888bf2.png)

Figure 7: The toolchain progresses from Scikit-learn to Decision Central, to Kie Server.

## 信任自动决策

一个决定越关键，你就越需要相信决定其结果的系统。一个次优的产品建议可能是可以接受的，但是拒绝贷款的决定或者关于医学发现的决定呢？此外，道德和法律要求我们在使用个人数据进行决策时承担责任。(例如，参见欧盟的[通用数据保护条例](https://gdpr-info.eu/)。)

### 检查

当在企业环境中引入自动决策系统时，通过监控一段时间内做出的决策来控制它是至关重要的。您应该能够使用决策管理技术中的工具来调查特定案例，并突出影响任何给定决策的特征。

通过 Red Hat Decision Manager，用户可以使用 Prometheus 和 Grafana 的公共监控堆栈来跟踪决策。通过分析 DMN 执行结果，您可以检查中间结果，并将它们与特定决策节点中捕获的企业策略相关联。

机器学习算法更加不透明:你得到输入数据和输出。从这个意义上说，机器学习模型是一个黑盒，不提供关于它如何工作的线索。专家将从算法参数中了解它的行为，但大多数业务用户无法访问这些信息。

### 使用知识背景

在我们的订单批准示例中，基于知识的元素是理解最终决策的关键。如果您可以看到手机的价格与模型中的参考价格相差甚远，您可以使用该信息来解释您的请求的决策结果。我们的模型很简单，所以结论很明显。对于复杂的模型来说，用知识背景包围机器学习算法甚至更有价值。拥有上下文有助于最终用户更好地理解决策结果。

**注**:未来，Red Hat Decision Manager 的开发团队将扩展其检查功能，以更好地应对 [TrustyAI](https://blog.kie.org/2020/06/trusty-ai-introduction.html) 挑战。

## 结论

在这篇由两部分组成的文章中，我们看到了人工智能不仅仅是机器学习。通过结合多种技术，我们可以增加机器学习模型的智能。此外，这种方法可以增加组织对机器学习结果的整体信心。业务用户和最终用户受益于知识环境提供的透明性。

我们为我们的示例项目制作了一个机器学习模型，然后我们从 DMN 模型中使用它。结果是一个“人工智能增强”的决定。然而，我们仅仅触及了人工智能的表面。如果你想更进一步，我推荐哈佛大学的这个免费课程: [CS50 用 Python 进行人工智能入门](https://cs50.harvard.edu/ai/2020/)。我们在本文中使用的 Python 示例基于课程中的一个类似示例。

我还发现 LinkedIn Learning(以前的 Lynda)上的[可解释的人工智能(XAI)课程](https://www.linkedin.com/learning/learning-xai-explainable-artificial-intelligence)非常有用。

## 承认

特别感谢我在工程团队中的同事:Edson Tirelli、Matteo Mortari 和 Gabriele Cardosi，他们为改进本文提供了建议和想法。Gabriele 还为本文撰写了“PMML 优势”一节。

*Last updated: January 20, 2021*