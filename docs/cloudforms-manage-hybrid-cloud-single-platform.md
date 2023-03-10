# CloudForms:通过单一平台管理您的 IT 和混合云

> 原文：<https://developers.redhat.com/blog/2017/11/09/cloudforms-manage-hybrid-cloud-single-platform>

在我开始谈论 IT 以及如何管理、控制和优化您的混合 IT 基础架构之前，我建议我们直接思考一下您的客厅，您通常在那里看电视、看电影、听音乐、玩视频游戏等。即使您不喜欢这种类型的娱乐，您也知道对于这些设备中的每一个，通常都使用遥控器来允许您在它们之间切换、管理它们以及控制所有您喜爱的节目。虽然这些设备正在融合为一体式架构，但它们是真正的多功能 。我们在很小的时候就已经学会了如何操作遥控器，这就是我们生活的现实。在这种情况下，您面对的是异构设备和各种遥控器，随着您购买新设备，遥控器的数量也会增加。管理一个简单任务的复杂性是很困难的，即管理您的日程，通过不同的控制操作多种设备，具有众多功能和不同的 供应商 。产品和 厂商 带来特定的特性，使用不同的命名法，并提供一些可能相互兼容也可能不兼容的特性。此外，由 厂商 提供的这些功能中的一些甚至在这些设备的整个生命周期中都不会被使用，这真是一种浪费！

![](img/6ce7a7114a125a67c4bdb1520213f02a.png)

图片 1 - 用多个遥控器管理多个设备

考虑到这种情况，您可能会想:处理各种娱乐设备和遥控器的复杂性与您的 IT 基础架构有什么关系？这与云计算或混合 IT 有什么关系？

## **传统与混合 IT**

![](img/50f7901b34e9b7395c26d413f0d8a959.png)  图片2-混合 IT 模式

我们几十年来所熟知的传统 IT 已经融合为一种模式，市场通常接受混合 IT。在这种新兴模式中，企业将继续使用裸机和虚拟化服务器来处理传统工作负载，但是，通过分别适用于 OpenStack 和 OpenShift 等产品的基础架构即服务(IaaS)和平台即服务(PaaS)技术，在私有云中使用 IT 服务。此外，这些公司将能够通过亚马逊或微软 Azure 等云提供商，在公共云中消费和托管其服务，将传统的 IT 资产收购模式转变为 IT 运营服务的消费模式。

我们谈论的是一个更加灵活的 IT 基础设施，准备好满足市场的新需求，并加强数字化转型，公司正在发展成为真正的数字化企业。这些数字业务需要一个灵活的 IT 模型，其资源可大规模扩展，以便云消费者可以按需使用计算资源，只为使用付费，并推动创新。下图描述了 Red Hat CloudForms 的功能。其功能旨在协同工作，为您的 混合 IT 提供强大的管理和维护。 ![](img/cb00318c442800cef4b8ed14f32e4928.png)

图片3-红帽 CloudForms 功能提供健壮的管理

## **技术市场的主要问题**

如今，技术市场的主要问题是业务线、IT 运营和开发人员之间的摩擦。企业关注的是市场的差异化，向客户提供更好的应用和服务，始终领先于竞争对手。另一方面，运营关注的是可用性、安全性、合规性和运营成本的降低。最后，开发人员希望在交付应用程序开发环境和投资创新方面具有敏捷性。通常，这两个群体之间存在分歧，因为开发人员希望自由地使用最新的功能，而不受技术的限制。 ![](img/091715abd1ddfa43f03d86952f832fc4.png)

图片4-IT 运营挑战与主导问题

尽管各地区之间存在差异和摩擦，但所有这些都是为了满足业务需求。这就是 CloudForms 可以帮忙的地方！

## ****什么是********cloud forms********又能有什么帮助呢？****

CloudForms 是一个混合 IT 的开源管理平台。该平台旨在管理小型或大型环境，并支持多个基础架构提供商、公共或私有云和容器。除了你的应用程序之外，你还可以管理你的本地基础设施，亚马逊、微软 Azure 或其他兼容的云提供商的实例。它可以自动执行任务，并创建可供开发人员按需使用的服务目录，所有服务都按部门、项目和配额进行细分。将通过单一界面进行访问和管理，提高运营效率，降低基础设施和企业应用交付的成本。CloudForms 是基于上游项目 ManageIQ 开发的、、、、设备的格式可用于、、【红帽虚拟化】、、、VMware、OpenStack、Amazon、Microsoft Azure、Google Cloud 或通过容器镜像用于原子主机、、、 Openshift。查看 Red Hat 文档中支持的平台 。[【2】](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/)

![](img/615223af6dc419bab1b555bf4f89d39e.png)

图片5-cloud forms 集成与工作流

> **市场调查显示，到 2017 年，60%的首席信息官将不得不降低基础设施成本，以资助业务创新。关于私有云，47%的公司已经在生产中部署了两个或更多。此外，74%的公司将购买解决方案来管理混合云和下一代应用程序的交付。** [ [1](https://www.redhat.com/files/resources/ptbr-itwob-conventional-agile-it-infographic-INC0242254.pdf)

## ****cloud forms******:**

*   **自助服务门户:**创建最终用户和运营商可以使用的服务目录。缩短基础架构交付时间，提高运营效率。
*   **计费:**通过计费功能，了解您 IT 基础架构中每个组件的运营成本。按部门、项目等生成按存储容量使用计费/显示报告。
*   **管理和控制:**多租户支持、配额、批准周期的创建、实例生命周期管理和策略创建。
*   **性能分析**:从您的虚拟机、实例、容器和主机中捕获指标，并了解计算资源的使用和消耗情况。
*   **战略** **报告:**通过收集的指标，为资源优化和容量规划生成“规模适当”的报告。
*   **Ansible 支持:**cloud forms和 Ansible 的结合是自动化各种工作流任务的强大工具。
*   **安全性**:根据预先建立的合规性策略扫描您的虚拟机、主机和容器。
*   **:监控主机和实例的事件和日志，通过时间轴和各种可用的过滤器查看。**
***   **集成:** 系统管理、工具和流程、事件控制台、配置管理数据库(CMDB)、基于角色的管理(RBA)和 Web 服务。**

## ****重要特征或资源突出:**** 

 ***   **可扩展架构:**可以按区域、分区对环境进行分段，复制 VMDBs，保证数据冗余。在官方文档中了解更多关于模型架构的信息。
*   **运营效率:** IT 基础设施管理通过单一的 web 界面集中管理。复杂性降低，IT 员工有更多时间投入到组织的战略活动中。
*   **无代理**:不需要安装代理来管理虚拟机、实例、容器和主机。
*   **可定制的 Web 界面:**为您的团队创建带有战略信息的定制仪表板，包括图表、绩效报告和事件。自定义登录界面并添加您的公司徽标。
*   **可用 REST API:**通过一个 REST API，开发人员可以轻松地将 ManageIQ 与其他企业应用集成，方便系统管理员通过脚本执行任务的自动化。

Red Hat 认为，开放的混合云为企业提供了灵活性、控制力和选择——并让您对创新敞开大门。它充分利用您的现有资源，同时允许系统、应用程序和数据跨公共云或私有云无缝协作。Red Hat 是唯一一家能够提供真正开放的混合云技术的公司。[ [4]](https://www.redhat.com/en/blog/open-hybrid-cloud-red-hats-vision-for-the-future-of-it)

![](img/f5e1b54d4b52bd2a4e4f1ae35e658712.png) 图片 6 -红帽开放混合云愿景

开放混合云是我们对 IT 未来的愿景，它涵盖了我们所做的一切。Red Hat CloudfForms 将成为混合云之旅的关键工具。

## **参考文献** **:**

[1]它是传统的还是敏捷的？[https://www . red hat . com/files/resources/ptbr-itwob-conventi on-agile-it-infograph-Inc 0242254 . pdf](https://www.redhat.com/files/resources/ptbr-itwob-conventional-agile-it-infographic-INC0242254.pdf)

[2] CloudForms 官方文档:[https://access . red hat . com/documentation/en-us/red _ hat _ cloud forms/](https://access.redhat.com/documentation/en-us/red_hat_cloudforms/)

[3]掌握云表单自动化，Peter McGowan，O'Reilly，2016

[4]开放混合云:红帽对 IT 未来的展望:[https://www . Red Hat . com/en/blog/Open-hybrid-cloud-Red-hats-vision-for-the-future-of-IT](https://www.redhat.com/en/blog/open-hybrid-cloud-red-hats-vision-for-the-future-of-it)

* * *

**红帽移动应用平台** [**下载**](https://developers.redhat.com/products/mobileplatform/download/) **，可在** [**红帽移动应用平台**](https://developers.redhat.com/products/mobileplatform/overview/) **了解更多。**

*Last updated: November 8, 2017***