# 3 按应用程序缩放开发人员门户文档

> 原文：<https://developers.redhat.com/blog/2018/01/24/3scale-developer-portal-docs-per-application>

在任何开发人员门户中拥有 Swagger 文档对于开发人员了解如何使用 API 是非常重要的。然而，并非所有的开发者都在使用相同的应用程序。如何让开发人员只看到与他们相关的文档呢？幸运的是，对于 [3Scale 开发者门户](https://www.3scale.net/api-management/api-developer-portal/)来说，一些 JavaScript 魔法可以让这成为可能。

首先，您需要确保在每个应用程序的基础上定义了 swagger JSON。例如，以这 3 个 JSON 文件为例，确保它们被发布在网络上的某个地方，没有页眉、页脚等。

一种简单的方法是导航到 3Scale 管理控制台中的“开发人员门户”选项卡，然后选择“新建页面”。给你的页面命名，并给它一个类似于`/<applicationName>.json`的路径。还要确保没有选择布局。然后保存并发布页面。**注意:**您可能需要转到草稿页面来发布页面。对所有应用程序都这样做。如果你愿意，你也可以在任何外部站点上托管这些。

一旦创建了所有的 swagger JSON 页面，您只需使用 JavaScript 找出用户有哪些应用程序，然后显示每个应用程序的 swagger 文档。从 3Scale 管理控制台内的开发人员门户配置页面，编辑文档页面，如下所示:

重要注意事项:

*   active docs 标记必须存在，以确保加载了 swagger 库。但是，不需要在标记中指定服务名。
*   必须为每个应用程序定义不同 id 的`div`。如果不这样做，只有最后一个应用程序的 Swagger UI 会加载，因为它会不断被覆盖。
*   您可以对这段代码使用任何想要的布局。
*   根据你的喜好调整日志。

* * *

了解更多关于 [Red Hat 3scale API 管理的信息](https://www.redhat.com/en/technologies/jboss-middleware/3scale)

*Last updated: September 3, 2019*