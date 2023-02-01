# 用于 Quarkus 智能应用的 Kogito

> 原文：<https://developers.redhat.com/blog/2019/08/29/kogito-for-quarkus-intelligent-applications>

Quarkus 项目在开发者中变得非常受欢迎。Quarkus 提供了一个快速开发环境，它已经有了一套库、标准和框架，可以通过 RestEasy、Panache、SmallRye、Keycloak 和 Kafka 等[扩展](https://quarkus.io/extensions/)获得。此外，您现在就可以开始使用 [Kogito](https://developers.redhat.com/blog/2019/07/23/devnation-live-introducing-kogito/) 来创建智能 Quarkus 应用程序。

[](/sites/default/files/tushuswatrutenaprog)Kogito models its logo after Odin, the Norse god who traded his own eye in exchange for wisdom.">

商业应用都是关于知识自动化的。jBPM 和 [Drools](https://www.drools.org/) 社区知道这一点，并希望为 Quarkus 开发者提供一种构建智能应用的方式。

Kogito 是一个开源的 Quarkus 扩展，允许开发者以一种更加业务驱动的方式实现核心逻辑。它带来了来自 jBPM 等产品测试项目 15 年以上经验的概念和成熟。

Kogito 是智能 Quarkus 应用程序的适当扩展。Kogito 预编译业务资产(例如，像 BPMN 文件或规则决策表)。它自动生成带有自己的 REST 端点的本机可执行文件，允许与各自的流程、任务和规则进行交互。这样，开发人员只需要担心逻辑本身的实现。

有了 Kogito，交付智能云原生业务应用将比以往任何时候都更容易。为了理解 Kogito 的强大，让我们快速看一下 jBPM 这样的业务自动化工具提供了什么。

### Red Hat 流程自动化经理

[红帽流程自动化管理器](https://developers.redhat.com/products/rhpam/download) (RHPAM)是 jBPM 项目的企业版，包含 Drools 和 OptaPlanner。RHPAM 以通过 BizDevOps 文化交付云就绪解决方案而闻名。它有一个使用商业和技术的可理解语言创作业务逻辑的生产环境:逻辑是按照三重模式实现的——[业务流程模型和符号](https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.1/html/administration_and_configuration_guide/chap-business_process_model_and_notation)(BPMN)；[案例管理模型和符号](https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager/7.3/html/designing_and_building_cases_for_case_management/case-management-cmmn-con-case-management-design) (CMMN)，以及[决策模型和符号](https://access.redhat.com/documentation/en-us/red_hat_process_automation_manager/7.0/html/designing_a_decision_service_using_dmn_models/dmn-con_dmn-models) (DMN)。

它还配备了一个强大的执行引擎 Kie Server，可以在混合云环境中轻松扩展，并可以通过 CI/CD 管道提供解耦的业务逻辑。要了解 RHPAM 这样的业务自动化工具的更多功能，请参考本文:[好消息:业务自动化与 SOA 无关](https://developers.redhat.com/blog/2019/02/20/good-news-business-automation-is-not-about-soa/)。

### Kogito ergo automate

使用 Kogito，开发人员可以在 Quarkus 上预编译和运行业务资产，在开发阶段利用热重载。此外，当使用本机模式编译项目时，业务规则的执行速度提高了 100 倍，并且消耗的资源更少。

不用说，两种模式下的启动时间都很棒(应用程序在启动后*准备好*进行访问；在应用首次访问时不会发生额外的处理)。查看这些 Kogito 示例项目的启动时间:

![](img/1b616efb99020c60b06e2c528ce72f3a.png)
Startup using GraalVM takes 1s. ![](img/b584fc3977e90c20e4650bb7b15c3ce3.png)
Startup in native mode, 0.007s.

请记住，要测试本机模式，您必须在您的机器上设置 GraalVM。还有，Kogito 还处于早期，0.3.0 版本将于本周(8 月)发布。

了解了业务自动化的未来，建议你试试扩展，加入 Kogito 社区！![?](img/3662ff39ae66cc5b6924cc0e12dabe7a.png)

### 了解更多关于 Kogito 的信息

要亲自测试 Kogito 并了解更多关于该项目的信息，请参考:

*   [Kogito 入门](https://kogito.kie.org/get-started/)
*   [科吉托语例句](https://github.com/kiegroup/kogito-examples)

从工程角度理解 Drools 到 Kogito 的迁移过程。在这个视频中，红帽首席软件工程师[马里奥·富斯科](https://developers.redhat.com/blog/author/mfusco/)也展示了 Kogito 的性能和实现:

*   [开发现场:介绍 Kogito](https://developers.redhat.com/blog/2019/07/23/devnation-live-introducing-kogito/)

这里是一个围绕 Kogito 开发的讨论论坛:

*   [科吉托谷歌集团](https://groups.google.com/forum/#!forum/kogito-development)

*Last updated: August 28, 2019*