# 宣布 Red Hat 应用程序迁移工具包 4.1.0:现在有技术报告

> 原文：<https://developers.redhat.com/blog/2018/07/25/announcing-red-hat-application-migration-toolkit-4-1-0-now-with-technical-reports>

[如果你没有关注 [Red Hat JBoss 中间件博客](https://middlewareblog.redhat.com/)，我们将[这篇文章](https://middlewareblog.redhat.com/2018/07/17/introducing-technical-reports/)转发到 developers.redhat.com 上]

[Red Hat Application Migration Toolkit](https://developers.redhat.com/products/rhamt/overview/)(RHAMT)4 . 1 . 0 已经发布，随之发布的还有一个我想在本文中重点介绍的新特性——技术报告。

如果您不熟悉 RHAMT，请查看我的前一篇文章,这篇文章介绍了 RHAMT，并描述了如何通过分析您的代码库，使用它来帮助将现有应用程序迁移到现代应用程序平台。

## 技术报告

RHAMT 中的这个新特性为所分析的应用程序提供了按功能分组的所用技术的汇总列表。它展示了技术是如何分布的。执行分析后，使用此报告可以快速比较数百个应用程序。此外，还会显示每个应用程序的大小、库数量和故事点总数，使您能够从单个报告中快速确定每个应用程序的类型，例如:

[![](img/7743a937d79c8a2466d4407307f0a2d9.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/07/technology-report-overview.png)

检查上面列表中的 application_13，我们可以看到这很可能是一个安全的前端应用程序，具有性能缓存。它包含几个库，其中大部分都与某种形式的安全性有关。

可以进一步检查每个应用程序，以确定使用的技术。例如，深入研究 application_13 会发现:

[![](img/3b632c7626900fdc993ff54e59b32806.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/07/technology-report-application.png)

在这里，我们可以看到每个类别中正在使用的精确库。如前所述，这个应用程序使用了大量的安全库，我们可以精确地识别所使用的技术。

无论您如何使用技术报告功能，我相信它对您的迁移和现代化工作都很有用。

*Last updated: November 15, 2018*