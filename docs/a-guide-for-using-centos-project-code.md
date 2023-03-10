# CentOS 项目代码使用指南

> 原文：<https://developers.redhat.com/blog/2021/02/03/a-guide-for-using-centos-project-code>

许多人向我们询问我们将如何发布 CentOS 源，以及我们是否会因为 2020 年 12 月 8 日的[公告](https://blog.centos.org/2020/12/future-is-centos-stream/)而做出改变，我们将专注于 CentOS 流。简而言之，我们不会对该流程进行任何更改。

CentOS 源仍将在包含 dist-git 样式源的存储库中发布到 git.centos.org。如果您正在寻找代码的公共访问，这将是一个好去处。

如果您考虑在自己的项目中使用这些代码——特别是如果该项目的目标是生产 Linux 发行版——我们已经整理了一些指南来帮助您遵守 Red Hat 服务协议和商标指南。本指南并不详尽。

本着社区协作的精神，我们在下面提供了一个做与不做的列表。

# 做

*   从上游或 git.centos.org 获得您的源代码，并遵循[红帽商标指南](https://www.redhat.com/en/about/trademark-guidelines-and-policies)。
*   明确描述你的发行是你做的东西，而不是红帽。您可能希望说您从特定位置的源代码开始，例如“根据取自 git.centos.org 的源代码修改和构建”
*   在发布或推广您的发行时，显著地包括以下免责声明:“Red Hat 和 CentOS 是 Red Hat，Inc .或其子公司在美国和其他国家的商标或注册商标。我们不隶属于、支持或赞助红帽或 CentOS 项目。”
*   遵守 GPL 和所有其他适用于您的构建的开源许可证。
*   如果您与 Red Hat 有协议，例如成为 Red Hat 开发者计划的成员或为 Red Hat 客户或合作伙伴工作，请查看协议条款，以便了解您的义务([https://www.redhat.com/en/about/agreements](https://www.redhat.com/en/about/agreements)和[个人开发者计划](https://www.redhat.com/wapps/tnc/viewterms/72ce03fd-1564-41f3-9707-a09747625585?extIdCarryOver=true&sc_cid=701f2000001Css0AAC))。

# 不

*   使用 Red Hat 订阅服务来创建或支持您的项目。
*   在项目、您的分销或您的项目推广和营销中使用任何 Red Hat 商标，包括 Red Hat 徽标，除非 [Red Hat 的商标指南](https://www.redhat.com/en/about/trademark-guidelines-and-policies)允许。