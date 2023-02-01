# 在 Red Hat OpenShift 上自动从 JBoss AMQ 6 迁移到 Red Hat AMQ 7

> 原文：<https://developers.redhat.com/blog/2019/05/01/automated-migration-from-jboss-a-mq-6-to-red-hat-amq-7-on-red-hat-openshift>

自从 Red Hat open shift Container Platform 首次发布以来，Red Hat 中间件产品就被提供用于在其上部署，并帮助开发人员构建更复杂的解决方案。消息代理是大多数新应用架构中非常重要的一部分，比如[微服务](https://developers.redhat.com/topics/microservices/)、[事件源](https://microservices.io/patterns/data/event-sourcing.html)和 [CQRS](https://microservices.io/patterns/data/cqrs.html) 。[红帽 JBoss AMQ](https://access.redhat.com/documentation/en-us/red_hat_jboss_a-mq/6.1/html/product_introduction/fusembintrowhatismb) 从一开始就提供在[红帽 OpenShift](https://developers.redhat.com/products/openshift/overview/) 上轻松部署消息代理。

[红帽 AMQ](https://www.redhat.com/en/technologies/jboss-middleware/amq) 7 是基于 [Apache ActiveMQ Artemis](http://activemq.apache.org/components/artemis/) 开源项目的高性能、可伸缩、多协议代理的最新版本。它还可以作为容器化的映像用于 Red Hat OpenShift，因此它允许开发人员在云环境中快速部署消息传递代理。

有一个[红帽 AMQ 7 迁移指南](https://access.redhat.com/documentation/en-us/red_hat_amq/7.2/html-single/migrating_to_red_hat_amq_7/index)有最重要的话题；但是，如何在红帽 OpenShift 上做到这一点并没有完全涵盖。根据我与客户的现场经验，我们在 OpenShift 上定义了一个自动化流程来完成这项工作，包括:

*   消费者和生产商的零停机时间
*   将红帽 JBoss AMQ 6 的任何持久消息移动到红帽 AMQ 7
*   使用可翻译的行动手册实现自动化

## 迁移过程

红帽 JBoss AMQ 6 (AMQ 6)提供了一个 drainer pod，只对持久模板可用，负责管理消息的迁移。当使用多节点配置部署 AMQ 6 时，当 AMQ 6 集群缩减时，消息可能会留在 KahaDB 存储目录中。为了防止消息被保留，drainer pod 将恢复它们并将它们移动到其他 AMQ 6 pod 实例。在幕后，一个经纪人的[网络在排水舱和其他 AMQ 6 舱之间建立起来。](http://activemq.apache.org/networks-of-brokers.html)

红帽 AMQ 7 提供了与先前版本相同的协议。drainer pod 使用 OpenWire 协议来移动消息，因此我们将使用它来将消息移动到新的红帽 AMQ 7 代理。

我们需要更改每个 AMQ 6 协议服务(openwire、amqp、stomp、mqtt)的服务选择器，以识别新的红帽 AMQ 7 pod 实例。当 AMQ 6 代理缩减时(副本= 0)，drainer pod 将移动持久化的消息。

此功能将非常简单的迁移过程定义为:

1.  扩大红帽 AMQ 7 经纪人，并等待准备就绪。
2.  修补 AMQ 6 服务，以平衡与红帽 AMQ 7 代理吊舱的连接。
3.  缩小 AMQ 6 排水器(*稍后启动*)。
4.  缩减 AMQ 6 号经纪人以避免新的联系。与新的红帽 AMQ 7 经纪人建立了新的联系。
5.  扩大 AMQ 6 排水器排水消息给新的红帽 AMQ 7 经纪人。
6.  检查 AMQ 6 排水器得出两个迁移周期。如果两个周期后没有要迁移的消息，我们可以确认 drainer 成功完成。
7.  按比例缩小 AMQ 6 排水器。

## 翻译剧本

上面描述的迁移过程包括步骤和检查，非常容易手动执行。然而，如果您需要为部署在 Red Hat OpenShift 中的每个 AMQ 6 代理执行此操作，那么可能会更加困难。角色剧本和[角色](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)可以帮助你轻松实现自动化。

我开发了一个简单可行的剧本，使用[命令模块](https://docs.ansible.com/ansible/2.4/command_module.html)和 [OpenShift CLI](https://docs.openshift.com/container-platform/3.11/cli_reference/get_started_cli.html) 来迁移代理。当然，还有其他替代方案来实现其他方法，比如， [oc 模块](https://docs.ansible.com/ansible/2.4/oc_module.html)、 [openshift_raw 模块](https://docs.ansible.com/ansible/2.5/modules/openshift_raw_module.html)、 [openshift 模块](https://docs.ansible.com/ansible/2.5/plugins/lookup/openshift.html)等。

这个可行的剧本样本可以在这个 [GitHub 库](https://github.com/rmarting/migration-amq6-amq7-openshift)中找到。

## 结论

这篇文章展示了在 Red Hat OpenShift 上将 Red Hat JBoss AMQ 6 代理迁移到 Red Hat AMQ 7 代理是多么容易。不要错过在您的 Red Hat OpenShift 部署中获得 Red Hat AMQ 7 最佳改进的机会。

*Last updated: September 3, 2019*