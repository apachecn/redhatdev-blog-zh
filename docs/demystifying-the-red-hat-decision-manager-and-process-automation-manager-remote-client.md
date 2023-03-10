# 揭开 Red Hat 决策管理器和过程自动化管理器远程客户端的神秘面纱

> 原文：<https://developers.redhat.com/blog/2018/12/10/demystifying-the-red-hat-decision-manager-and-process-automation-manager-remote-client>

KIE 服务器是[红帽决策管理器](https://developers.redhat.com/products/red-hat-decision-manager/overview/) (RHDM)和[红帽流程自动化管理器](https://developers.redhat.com/products/rhpam/overview/) (RHPAM)平台的轻量级、云原生、规则和流程执行运行时。最近，我收到越来越多关于如何使用 KIE 服务器客户端 Java API 与 KIE 服务器执行运行时 [RHDM](https://developers.redhat.com/products/red-hat-decision-manager/overview/) (以前叫红帽 JBoss BRMS)和 [RHPAM](https://developers.redhat.com/products/rhpam/overview/) (RHPAM)进行交互的问题。为了回答这些问题，并创建一个将来的参考，我决定写一些代码示例，并附上这篇文章。

KIE 服务器客户端 Java API 为 Java 应用程序提供了一种与 RHDM 和 RHPAM 的 KIE 服务器执行引擎通信的简单方法。API 将应用程序从底层的 REST 和/或 JMS 通信协议和传输中抽象出来，使得与服务器的集成更容易构建、测试和维护。

## 什么是 KIE 服务器客户端？

KIE 服务器客户端是一个小型的轻量级 Java 库，提供了一个纯 Java API，允许 Java 客户端通过 RESTful(或 JMS)通信通道与 Red Hat Decision Manager 和 RHPAM 执行服务器进行交互。这将客户端从将请求和响应数据封送到执行服务器期望的格式(JSON、JAXB 或 XSTREAM)中抽象出来，并通过 HTTP(或使用 JMS API 时通过 JMS 代理)处理发送请求和接收响应。这极大地简化了想要与我们的平台交互的 Java 客户端的代码。

## API

我们先来看看 API 本身。API 是围绕`KieServicesClient`构建的。`KieServicesClient`是可以从`KieServicesFactory`构建的主要组件之一，它让我们可以访问 KIE 服务器提供的各种运行时。KIE 服务器使用基于插件的系统，并且平台的每个功能都是通过插件提供的。

对于每个公开远程接口的插件，客户机提供一个匹配的客户端服务。例如，为了通过 KIE 服务器规则引擎执行规则，我们使用`RuleServicesClient`，决策模型和符号(DMN)模型用`DMNServicesClient`评估，与流程引擎的交互通过`ProcessServicesClient`完成。其他可用的客户端有:

*   `CaseServicesClient`与案件管理系统交互
*   `UserTaskServicesClient`与流程引擎的用户任务引擎交互
*   `QueryServicesClient`对服务器执行查询
*   `SolverServicesClient`集成 OptaPlanner 功能

还有各种管理客户端。

下面的代码显示了如何从主`KieServicesClient`中检索特定的客户端，在本例中是一个`ProcessServicesClient`:

```
KieServicesClient kieServicesClient = KieServicesFactory.newKieServicesClient(kieServicesConfig);
ProcessServicesClientprocessClient=kieServicesClient.getServicesClient(ProcessServicesClient.class);
```

在可能的情况下，各种远程客户端 API 非常接近 KIE、Drools、Red Hat JBoss Frameworks(以前的 Red Hat JBoss jBPM)和 OptaPlanner 公开的标准 Java APIs。但是，用户需要注意一些不同之处。

## 使用正确的 KIE 会话

当使用 Decision Manager/Drools 作为嵌入式库时，从一个`KieContainer`中检索一个`KieSession`，即您插入数据和执行规则的会话。**`KieContainer`API 允许你选择你想要使用的特定`KieSession`。不同的`KieSession`可以在规则部署单元(KJAR)的`kmodule.xml`部署描述符中配置。这允许我们定义和配置不同种类的会话，例如，使用伪时钟的无状态`KieSession`会话、有状态`KieSession`会话或`KieSession`会话——无论我们的用例需要什么。**

 **当使用远程 KIE 服务器客户端时，选择正确的`KieSession`是不同的，因为 API 不公开`KieContainer`。相反，我们需要指定在`BatchExecutorCommand`中使用的`KieSession`的“ID”，如下所示:

```
BatchExecutionCommand batchExecutionCommand = commandFactory.newBatchExecution(commands, "default-stateless-kiession");
```

这将产生一个包含“查找”条目的 JSON 有效负载，它告诉 KIE 服务器给定请求使用哪个会话:

```
{
  "lookup" : "default-stateless-ksession",
  "commands" : [ {
```

ID 应该与 KJAR 的`kmodule.xml`中定义的`KieSession`的 ID 相匹配，如下所示:

```
<kmodule  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <kbase name="default-kbase" default="true" eventProcessingMode="cloud" equalsBehavior="equality">
    <ksession name="default-stateless-ksession" type="stateless" default="true" clockType="realtime"/>
  </kbase>
</kmodule>
```

当使用远程客户端 API 时，建议使用无状态的`KieSession`,除非您的用例明确要求有状态会话。

## 封送处理和自定义数据类型

当使用远程 API 时，数据需要以双方都能理解的格式从客户机发送到服务器。KIE 服务器支持三种格式:JSON、JAXB 和 XSTREAM。一些 KIE 服务器 API，比如启动业务流程的 API，接受通用数据类型作为输入，比如一个`Map`。这个`Map`用于向流程执行引擎发送流程开始变量。

当我们的流程变量包含自定义的数据类型时，例如，`Applicant`或`Loan`，我们需要向 KIE 服务器客户端注册这些数据类型。这允许编组器组件在与远程 KIE 服务器通信时正确地编组和解组这些数据类型。Java 类可以简单地通过`KieServicesConfiguration`注册，如下所示:

```
KieServicesConfiguration kieServicesConfig = KieServicesFactory.newRestConfiguration(KIE_SERVER_URL, credentialsProvider);
Set<Class<?>> extraClasses = new HashSet<>();
extraClasses.add(Application.class);
extraClasses.add(Applicant.class);
extraClasses.add(Property.class);
kieServicesConfig.addExtraClasses(extraClasses);
```

## 例子

在这个 GitHub 库中，我收集了三个与 KIE 服务器上的远程服务通信的 KIE 服务器客户端的例子。这些示例显示了如何:

*   启动业务流程。
*   执行规则。
*   评估一个 DMN 模型。

提供了三个`Main`类来演示 API 的用法。代码中的注释进一步解释了所使用的 API。

## 结论

本文简要介绍了 KIE 服务器客户端 Java API，提供了一些如何最好地使用该 API 的提示和技巧，并以演示如何启动一个进程、执行规则和评估 DMN 模型的示例作为结束。

以下是您可能感兴趣的相关文章:

*   [在您的云中快速尝试 Red Hat Decision Manager](https://developers.redhat.com/blog/2018/11/19/try-red-hat-decision-openshift/)
*   [在您的云中快速试用 Red Hat Process Automation Manager](https://developers.redhat.com/blog/2018/12/04/quickly-try-red-hat-process-automation-manager-in-your-cloud/)
*   [借助 Red Hat Process Automation Manager 实现 Spring Boot 业务流程自动化](https://developers.redhat.com/blog/2018/11/01/spring-boot-enabled-business-process-automation-with-red-hat-process-automation-manager/#more-523727)

![Duncan Doyle](img/31cf5c8a0bf8e97ac6b3ae99ebeaad6f.png)

### 关于作者:

[Duncan Doyle](http://twitter.com/DuncanDoyle) 是 Red Hat 决策管理器和流程自动化管理器平台的技术营销经理。Duncan 拥有 Red Hat 咨询和服务的背景，曾与大型 Red Hat 客户广泛合作，构建高级、开源、业务规则和业务流程管理解决方案。

他在面向服务的架构、持续集成和交付、规则引擎和 BPM 平台等技术和概念方面有着深厚的背景，是多种 JBoss 中间件技术的主题专家(SME ),包括但不限于 JBoss EAP、HornetQ、Fuse、DataGrid、BRMS 和 BPMSuite。当他不从事开源解决方案和技术时，他就和他的儿子和女儿一起建造乐高，或者在他的 Fender Stratocaster 上播放一些 90 年代的摇滚音乐。

*Last updated: May 30, 2022***