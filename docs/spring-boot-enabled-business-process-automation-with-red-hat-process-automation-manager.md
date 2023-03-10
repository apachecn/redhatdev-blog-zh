# 借助 Red Hat Process Automation Manager 实现 Spring Boot 业务流程自动化

> 原文：<https://developers.redhat.com/blog/2018/11/01/spring-boot-enabled-business-process-automation-with-red-hat-process-automation-manager>

随着 7.1 版[Red Hat Process Automation Manager](https://developers.redhat.com/products/rhpam/overview/)(rhp am)的发布，该平台现在支持将 Process Automation Manager 运行时部署为 Spring Boot 应用中的一项“功能”。正如 jBPM.org(RHPAM 的上游社区项目)[的项目负责人 Maciej Swiderski 在今年早些时候](http://mswiderski.blogspot.com/2018/01/spring-boot-starters-for-jbpm-and-kie.html)解释的那样，rhp am 构建于其上的 KIE(知识就是一切)平台为 Spring Boot 创业者*提供了使用最少的代码快速构建具有流程和案例执行能力的业务应用程序或[微服务](https://developers.redhat.com/topics/microservices/)。*

Spring Boot 启动器由一组依赖描述符组成，可以添加到您的应用程序中，以便为您的项目轻松设置正确的依赖关系。RHPAM 现在为以下五个初学者提供支持。这使您能够灵活选择您在 Spring Boot 应用程序中所需的流程自动化功能:

*   `jbpm-spring-boot-starter-basic`:为您的应用程序添加 [jBPM](http://www.jbpm.org) 功能。它提供了 [jBPM](http://www.jbpm.org) /RHPAM 嵌入式运行时和服务，以在您的应用程序中运行流程执行引擎。
*   `kie-server-spring-boot-starter`:为您的应用程序添加 KIE 服务器功能。这允许您在应用程序中运行 KIE 服务器服务和 RESTful APIs。这个启动器提供了 Process Automation Manager 必须提供的所有业务自动化特性。这包括流程和案例执行( [jBPM](http://www.jbpm.org) )、(业务)规则和 DMN(决策模型&符号)执行( [Drools](http://www.drools.org) )，以及业务资源优化( [OptaPlanner](http://www.optaplanner.org) )。
*   `kie-server-spring-boot-starter-drools`:和`kie-server-spring-boot-starter`一样，但是只有规则和 DMN 执行([口水](http://www.drools.org))能力。
*   `kie-server-spring-boot-starter-jbpm`:与`kie-server-spring-boot-starter`相同，但是只有规则、流程和案例执行(jBPM)功能。
*   `kie-server-spring-boot-starter-optaplanner`:同`kie-server-spring-boot-starter`，但只有业务资源优化( [OptaPlanner](http://www.optaplanner.org) )能力。

## 云原生流程自动化功能

KIE·Spring Boot 的启动者使在 IT 架构中部署业务自动化功能的新的和有趣的方法成为可能。在今天的云和容器世界中，业务流程执行开始从流程执行引擎的传统集中式部署(来自 SOA 时代)转变为更敏捷、更小流程定义的非集中式部署。这些较小的流程部署非常适合[微服务](https://developers.redhat.com/topics/microservices/)架构。在这些架构中，它们可以编排微服务，成为微服务编排的一部分，或者两者兼而有之。

下图显示了如何将 jBPM 流程执行引擎部署为微服务架构中的一项功能的高级概述。在这个例子中，*订单服务*和*运输服务*都使用 jBPM 的流程执行能力，而*定价服务*和*促销服务*使用 Drools 的规则执行能力。

[![High-level overview of how the jBPM process execution engine can be deployed as a capability within a microservices architecture](img/65edf767626212b1cce5029ba97947a4.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/Screenshot-2018-10-03-at-17.17.56.png)

需要轻量级的运行时和部署模型，能够在现代容器平台上以云本地的方式运行这些流程实例，例如 [Red Hat OpenShift](http://openshift.com/) 。Process Automation Manager 版已经通过 OpenShift KIE 服务器映像为 OpenShift 部署提供了全面支持。7.1 版本为开发人员社区提供了更大的灵活性，允许开发人员将高级业务自动化功能集成到他们的 Spring Boot 应用和微服务中，并将其部署在现代 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 集群上。这使得开发更小、更轻的服务成为可能，这些服务具有更高级的功能来自动化业务流程执行、业务规则执行和评估以及业务资源优化。

## 企业流程管理

随着更多用于自动化业务流程管理和执行的轻量级选项的引入，仍然需要提供企业级管理功能和对这些自动化流程的洞察。毕竟，这些过程实现了组织的价值链，因此需要被有效地管理。因此，KIE 服务器启用的 Spring Boot 运行时与 KIE 服务器控制器和 KIE 服务器智能路由器集成，从而能够轻松地与流程自动化管理器业务中心工作台集成。这使得最终用户能够从一个集中的(或分布式的！)管理控制台。

[![KIE-Server capability deployed in a Spring Boot application](img/aca37c86b84214e84cb4827b29586217.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/10/Screenshot-2018-10-03-at-17.52.35.png)

从上面的屏幕截图中可以看出，部署在 Spring Boot 应用程序中的 KIE 服务器功能在业务中央管理控制台中注册为“托管的”KIE 服务器运行时。这使得系统管理员和操作员能够高效和有效地管理企业流程运行时，即使是在分布式、云本地和容器化的运行时拓扑中。

## 演示

在 [JBoss 演示中心 GitHub 存储库](https://github.com/jbossdemocentral/rhpam7-order-it-hw-demo)中的新*订单 IT 硬件*演示中提供了这一新功能的演示。该应用程序演示了一个基于 Process Automation Manager 7 的案例管理功能构建的 IT 硬件订单系统。流程引擎和 KIE 服务器嵌入在 Spring Boot 应用程序中运行，由 RHPAM Business Central workbench 管理。此外，演示了与 Vert.x 微服务的 RESTful 集成，以及 BPMN2 中的[传奇模式](https://microservices.io/patterns/data/saga.html)的实现。

Spring Boot 应用程序的代码可以在[这里](https://github.com/jbossdemocentral/rhpam7-order-it-hw-demo-springboot-app)找到。[项目的 Maven POM 文件中的这个依赖关系](https://github.com/jbossdemocentral/rhpam7-order-it-hw-demo-springboot-app/blob/master/pom.xml#L31-L35)显示了 KIE 服务器 Spring Boot 启动器的配置。只需通过 [@SpringBootApplication](https://github.com/DuncanDoyle/order-it-hw-app/blob/master/src/main/java/org/jbpm/cases/orderithwapp/OrderItHwAppApplication.java#L15) 注释将应用程序标记为 Spring Boot 应用程序，KIE 服务器的功能就可以在应用程序内部进行引导。

该演示包括一个完整的演练指南，提供了关于如何在 OpenShift 上设置演示并运行完整的端到端用例的完整说明。

## 结论

随着 Red Hat Process Automation Manager 7.1 的发布，Spring Boot 现代轻量级应用程序的开发人员现在能够通过新的 KIE·Spring Boot 启动器，用业务流程和业务规则执行功能来增强和扩充他们的应用程序。在 OpenShift 上的这些业务应用程序和服务的支持下，Red Hat Process Automation Manager 能够在现代容器平台上实现分布式流程和规则管理。同时，与 Business Central workbench 的集成提供了企业范围的功能，以便在分布式生产环境中有效地监控和管理这些业务流程和规则。

![Duncan Doyle](img/31cf5c8a0bf8e97ac6b3ae99ebeaad6f.png)

### 关于作者:

[Duncan Doyle](http://twitter.com/DuncanDoyle) 是 Red Hat 业务自动化平台的技术营销经理。Duncan 拥有 Red Hat 咨询和服务的背景，曾与大型 Red Hat 客户广泛合作，构建高级、开源、业务规则和业务流程管理解决方案。

他在面向服务的架构、持续集成和交付、规则引擎和 BPM 平台等技术和概念方面有着深厚的背景，是多种 JBoss 中间件技术的主题专家(SME ),包括但不限于 JBoss EAP、HornetQ、Fuse、DataGrid、BRMS 和 BPMSuite。当他不从事开源解决方案和技术时，他就和他的儿子和女儿一起建造乐高，或者在他的 Fender Stratocaster 上播放一些 90 年代的摇滚音乐。

*Last updated: September 3, 2019*