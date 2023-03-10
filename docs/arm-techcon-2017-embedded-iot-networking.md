# ARM TechCon 2017 -嵌入式、物联网、网络等...

> 原文：<https://developers.redhat.com/blog/2017/11/17/arm-techcon-2017-embedded-iot-networking>

**Arm TechCon 2017 -嵌入式、物联网、网络和无服务器聚焦**

上个月是 Arm TechCon，这是一年一度的开发者大会，展示了 Arm 及其合作伙伴的产品。Arm 展示了其愿景和战略，以实现处理器的更大集成，并绕过缓慢的摩尔定律。 和往常一样，有一群新产品发布，但总的来说，展会似乎缺乏过去几年的活力，尤其是去年 Arm 被软银收购后的兴奋。例如，没有像去年来自 的孙正义(软银董事长&首席执行官)那样的大愿景主题演讲，他谈到物联网使寒武纪大爆发成为可能(这使地球上成千上万个新物种成为可能)，在 20 年内导致 1 万亿个物联网设备。

考虑到 Arm 架构广泛的服务器使用案例，会议分为以下几个部分:

*   嵌入式软件开发
*   汽车、工业和功能安全
*   计算机视觉、机器学习和图形学
*   高效系统
*   物联网
*   联网&服务器
*   硅设计
*   信任&安全
*   Mbed 连接

**超越设备的思考**

Remi Pottier (Arm)做了一个有趣的演讲，内容是物联网超越设备，并通过物联网实现新的商业模式。讲座内容是价值链中各种参与者的角色，包括那些不常被提及的角色，如咨询/集成服务、开发运维、数据中心管理和监控工具。

在另一场有趣的会议中，Kinjal Dave (Arm)谈到了物联网的异构多处理。异构系统结合了不同的计算元素，例如 Arm BIG。很少结合 Cortex-A、Cortex-R 或 Cortex-M 内核来提供适当规模的处理。

在关于加速边缘智能的会议上，Govind Wathan (Arm)讨论了边缘计算的要求，如实时/延迟、可靠性、带宽/成本、安全性和隐私。他引用了物联网项目中物联网网关使用的统计数据，从目前的 60%增加到 2020 年的 90%。

继续边缘计算的主题，在我的会议中，我谈到了使用企业工具构建智能物联网网关。对于那些对构建智能物联网网关感兴趣的人，我介绍了使用 Red Hat Ansible 提供网关所需的步骤。它基于我在今年早些时候创建的网关演示，使用企业工具来构建 Arm 驱动的网关。这个 demo 的细节有 [这里有](https://github.com/redhat-iot/Virtual_IoT_Gateway/tree/revert-1-Arm-64-Platform) 。

**Arm 服务器厂商-行动中的缺失？**

与过去的几场 Arm TechCons 不同，Arm 驱动的服务器并没有引起太多的讨论。展会上也没有主要 Arm 服务器芯片供应商的展位:卡威姆、高通等。然而，这些供应商在会议期间提出了几个会议。在 Red Hat 展台，我们展示了智能物联网网关的演示，该网关基于 X-Gene2 驱动的 64 位 SoC，运行 Red Hat Enterprise Linux 和 Red Hat JBoss 中间件来处理消息传输和数据集成。

或许，Arm TechCon 已经变得有点无聊了，但无聊是件好事，因为这意味着随着技术的成熟，它正被越来越广泛地采用。一个关键要点是 Arm 如何继续在嵌入式、物联网、网络和其他领域实现路线图，以满足合作伙伴、客户和生态系统的期望。

* * *

[**加入红帽开发者计划**](https://developers.redhat.com/?intcmp=70160000000xZNgAAM) **(免费)并获得相关的备忘单、书籍和产品下载。**

*Last updated: November 15, 2017*