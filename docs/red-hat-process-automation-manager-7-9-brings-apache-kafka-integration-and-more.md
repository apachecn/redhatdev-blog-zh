# red Hat Process Automation Manager 7.9 带来了 Apache Kafka 集成等等

> 原文：<https://developers.redhat.com/blog/2020/11/27/red-hat-process-automation-manager-7-9-brings-apache-kafka-integration-and-more>

[Red Hat Process Automation Manager](https://developers.redhat.com/products/rhpam/overview)7.9 为流程和案例管理、业务和决策自动化以及业务优化带来了错误修复、性能改进和新功能。本文向您介绍了 Process Automation Manager 与 [Apache Kafka](https://developers.redhat.com/topics/kafka-kubernetes) 的现成集成，改进的业务自动化管理功能，以及对多决策需求图(drd)的支持。我还将指导您设置和使用新的`drools-metric`模块来分析业务规则性能，并且我将简要介绍一下在 Process Automation Manager 7.9 中的 [Spring Boot](https://developers.redhat.com/topics/spring-boot) 集成。

**注意**:Red Hat Process Automation Manager 7.9 与决策管理和业务优化相关的更新也适用于 Red Hat Decision Manager 7.9。

## 关于流程自动化管理器

红帽流程自动化管理器(以前的红帽 JBoss BPM 套件)和红帽决策管理器(以前的红帽 BRM 套件)基于久经考验的开源软件，如 [jBPM](https://www.jbpm.org/) 、 [Drools](http://drools.org/) 、 [OptaPlanner](https://www.optaplanner.org/) 和 [Kogito](http://kogito.kie.org/) 。图 1 是每个产品功能的概述。

[![The diagram shows Red Hat Decision Manager inside of Process Automation Manager and lists each tool's features and capabilities.](img/64ba380e67af17920f3130341e934c30.png "Screen Shot 2020-11-16 at 15.00.08")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.00.08.png)

Figure 1: Overview of Red Hat Process Automation Manager and Red Hat Decision Manager.

流程自动化管理器和决策管理器运行在内部和[容器化环境](https://developers.redhat.com/topics/containers)之上，利用企业编排平台，如 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 和各种云提供商。

## 将流程自动化管理器与红帽 AMQ 流(Kafka)集成

作为我们努力改进过程自动化管理器和红帽 AMQ 流 T2 之间的集成的一部分，我们在 7.9 版本中引入了一个新的 Kafka 任务。如图 2 所示，`WorkItemHandler`任务允许您向 Kafka 代理中运行的 Kafka 主题发出事件。

[![An illustration of the Kafka event emitter.](img/40471ad727592c6a60c0a69992c0bc78.png "Screen Shot 2020-11-16 at 15.00.57")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.00.57.png)

Figure 2: Emit an event to Kafka.

这个新的 Kafka 任务确保消息在到达下一个节点之前被传递到 Kafka 主题。该任务使用与处理其他任务中的错误相同的重试/重新引发引擎机制。

默认情况下,`WorkItemHandler`带有三个`String`输入数据赋值:一个事件`key`、`topic`名称和您想要在事件中发送的`value`。对于输出，任务使用默认的`String`类型`result`。

在使用`WorkItemHandler`之前，您需要通过在 Business Central 和您的项目中启用任务来设置您的环境。您还需要将任务作为 Maven 依赖项添加到项目中。要了解有关在商务中心使用`WorkItemHandler`任务的更多信息，请参见 [*在商务中心*](https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager/7.9/html-single/developing_process_services_in_red_hat_process_automation_manager/index#manage-service-tasks-proc_custom-tasks) 中管理自定义任务。

## 使用`drools-metric`模块分析规则性能

这个版本引入了一个新的`drools-metric`模块，用于识别业务规则中的性能瓶颈。一旦找到瓶颈，就可以调整规则实现来提高性能。

Drools 规则引擎使用[ph leak 算法](https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager/7.8/html/decision_engine_in_red_hat_process_automation_manager/phreak-algorithm-con_decision-engine)根据业务规则验证工作内存中的事实。在代码中调整业务规则语法可以显著提高 Phreak 的性能。

在下一节中，我将向您展示如何使用`drools-metric`模块和`ReteDumper`助手类来发现错误的语法。我们将使用来自[metricologutils](https://github.com/tkobayas/MetricLogUtils/wiki/How-to-use-MetricLogUtils)GitHub 仓库的一个例子来进行演示。

### 使用 ReteDumper 进行规则分析

首先，您可以使用`ReteDumper` helper 类输出每个 Phreak 节点花费的时间，这样您就可以看到一个节点被激活了多少次。图 3 中的截屏显示了一个`ReteDumper`输出的例子。

[![Console output from the ReteDumper class.](img/f2d1a2ecb9271721531fccc7899458cc.png "Screen Shot 2020-11-16 at 15.03.26")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.03.26.png)

Figure 3: ReteDumper output in the console.

通过记录该输出中的数据，您可以发现如图 4 所示的异常结果。

[![Console output with errors highlighted.](img/920648e9fdddd6b9cbfba8f785c5c9ec.png "Screen Shot 2020-11-16 at 15.04.10")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.04.10.png)

Figure 4: Use the output to identify rules associated with a slow Phreak node.

我用红色突出显示的`Collect expensive orders combination`规则与一个慢速 Phreak 节点相关联。用蓝色突出显示的`eval`条件在规则评估期间导致了太多的连接组合。结合性能调优，这种类型的数据分析可以将规则的执行时间从 1，524 微秒缩短到 112 微秒。让我们浏览一个来自[metricologutils](https://github.com/tkobayas/MetricLogUtils/wiki/How-to-use-MetricLogUtils)GitHub 仓库的用例，这样您就可以亲自看到这些结果。

### `drools-metric`的一个用例

这个例子展示了在一个常见用例中更新业务规则语法所带来的性能优势。本例中的文件可以在 MetricLogUtils 存储库的 [src/main/resources](https://github.com/tkobayas/MetricLogUtils/tree/master/use-cases/use-case-join) 文件夹中找到:

1.  克隆项目并进入其文件夹:

    ```
    $ git clone https://github.com/tkobayas/MetricLogUtils.git

    $ cd MetricLogUtils/use-cases/use-case-join/

    ```

2.  使用默认规则实现运行项目测试:

    ```
    $ mvn test

    ```

3.  检查日志中的结果。您应该会看到一行以微秒为单位的运行时间:`elapsed time (ms)`。另外，注意延迟时间:`AccumulateNode(8)`。
4.  检查文件`Sample.drl`和`Sample.drl.fix1`中规则实现的差异。你会在 MetricLogUtils `src/main/resources/com/sample/`文件夹中找到这些文件。
5.  为了提高规则性能，用`Sample.drl.fix1`中可用的实现替换文件`Sample.drl`的内容。
6.  重新运行测试并检查结果:

    ```
    $ mvn test

    ```

7.  对文件`Sample.drl.fix2`执行相同的步骤，注意性能的提高。

### 在决策管理器 7.9 中使用`drools-metric`

要在决策管理器 7.9 中使用新的`drools-metric`模块，您需要将`org.drools:drools-metric` Maven 依赖项添加到您的规则项目中。您还需要为类`org.drools.metric.util.MetricLogUtils`启用跟踪日志。[见*第二十章。决策管理器文档中的 DRL 红帽决策管理器 7.9*](https://access.redhat.com/documentation/en-us/red_hat_decision_manager/7.9/html/developing_decision_services_in_red_hat_decision_manager/performance-tuning-drl-ref_drl-rules) 的性能调整注意事项了解更多详细信息。

## 新的业务仪表板自动化管道

Process Automation Manager 7.8 引入了 Dashbuilder 组件，用于为业务仪表板部署独立环境。Dashbuilder 允许您在 Business Central 中创建业务仪表板，并在多个环境中部署报告。图 5 显示了一个显示实时收集的关键性能指标(KPI)的仪表板报告。

[![A graph displays KPIs in the dashboard.](img/3f7fc5991b72e4376b09ae0b6addc225.png "Screen Shot 2020-11-16 at 15.05.39")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.05.39.png)

Figure 5: A dashboard exposes KPIs collected in real-time.

从 Process Automation Manager 7.9 开始，您可以使用 REST APIs 或用户界面(UI)来管理仪表板。自动化管道允许您使用单个 Dashbuilder 组件导出、导入甚至部署多个仪表板。

我们还改进了报告定制。您现在可以[开发自己的组件](https://blog.kie.org/2020/09/developing-custom-components-for-dashbuilder.html)或者使用第三方组件来显示业务数据。图 6 中显示的热图组件是从 [Dashbuilder 社区](https://github.com/jesuino/dashbuilder-components)免费获得的众多组件中的一个例子。热图让我们一目了然地看到哪些节点被更频繁地触发，哪些节点需要更多的时间来执行。

[![A flow diagram including the Heatmap component.](img/635853b60475ecf0b36e76879a16ad77.png "Screen Shot 2020-11-16 at 15.06.28")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.06.28.png)

Figure 6: Heatmap is one of many components developed by the Dashbuilder community.

## DMN 模型中的多个 drd

您现在可以在您的决策模型符号(DMN)模型中使用多个决策需求图(drd)。通过将复杂的决策按照它们的领域进行划分，我们可以更好地组织决策逻辑。图 7 展示了使用多个 drd 将两组关键决策分解成两个更小的集合。

[![Two sets of decisions are broken into smaller subsets.](img/a134966af1ea37688e3b5ebc2f38a407.png "Screen Shot 2020-11-16 at 15.07.00")](/sites/default/files/blog/2020/11/Screen-Shot-2020-11-16-at-15.07.00.png)

Figure 7: Using multiple DRDs in a complex DMN model.

## 观看视频:DMN—过程自动化管理器 7.9 中的多 DRDs 支持

如下面的视频演示所示，您可以轻松地操作节点，并将它们移动到新的图或现有的图中。该视频还演示了如何导航图表、重命名图表，以及在一个自包含的 DMN 模型中模拟您的决策。

[https://www.youtube.com/embed/omw5apSc5gk?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/omw5apSc5gk?autoplay=0&start=0&rel=0)

## SpringBoot 集成改进

这个版本还通过增强审计数据复制和允许您创建自包含 JAR 部署，改进了 Process Automation Manager 与 [Spring Boot](https://developers.redhat.com/topics/spring-boot) 的集成。

## 结论

本文展示了 Red Hat Process Automation Manager 7.9 中许多新特性的示例。参见[流程自动化管理器 7.9](https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager/7.9/html/release_notes_for_red_hat_process_automation_manager_7.9/index) 和[决策管理器 7.9](https://access.redhat.com/documentation/en-us/red_hat_decision_manager/7.9/html/release_notes_for_red_hat_decision_manager_7.9/index) 的发行说明，了解 Spring Boot 集成、错误修复、安全改进等等

*Last updated: May 18, 2021*