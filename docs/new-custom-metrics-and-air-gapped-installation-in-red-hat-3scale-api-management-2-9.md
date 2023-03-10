# Red Hat 3scale API Management 2.9 中新的自定义指标和空气间隙安装

> 原文：<https://developers.redhat.com/blog/2020/10/29/new-custom-metrics-and-air-gapped-installation-in-red-hat-3scale-api-management-2-9>

我们继续更新 [Red Hat Integration](https://www.redhat.com/en/products/integration) 产品组合，为现代[云](https://developers.redhat.com/topics/serverless-architecture)和[容器](https://developers.redhat.com/topics/containers)原生应用提供更好的运营和开发体验。 [Red Hat Integration 2020-Q3 版本](https://access.redhat.com/documentation/en-us/red_hat_integration/2020-q3/html/release_notes_for_red_hat_integration_2020-q3/)包括[Red Hat 3scale API Management 2.9](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/release_notes_for_red_hat_3scale_api_management_2.9_on-premises/index)，为 3 scale 提供了新的特性和功能。在其他特性中，我们更新了 [3scale API 管理](https://developers.redhat.com/products/3scale/overview)和网关操作符。

本文介绍了 Red Hat 3scale API Management 2.9 发布亮点，包括在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 上的 3scale 的气隙安装，以及针对自定义指标和上游相互传输层安全性(TLS)的新 APIcast 策略。

**注**:关于 Red Hat 赞助的 APIDays LIVE London 的注册信息，请参见文章末尾:使用 API 实现嵌入式金融、银行和保险之路。注册是免费的，包括两个录制的红帽会议。

## OpenShift 上的气隙安装

3scale [操作符](https://developers.redhat.com/topics/kubernetes/operators)现在完全支持 OpenShift 上 [3scale API 管理的气隙安装。*空气间隙*或受限网络与互联网隔离，并且在物理上与任何其他网络隔离。政府机构和金融机构等安全环境通常需要在 OpenShift 上安装气隙式 Red Hat 集成。这种类型的安装在三个方面不同于常规安装:](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/installing_3scale/index#installing-configuring-threescale-operator-using-olm)

*   OpenShift 软件频道和存储库不能从 Red Hat 的内容分发网络获得。
*   您不能直接从 Red Hat 的容器注册表中提取 OpenShift 图像。
*   你不能连接到红帽托管的 Maven 镜像。

为了允许在受限网络上安装，3scale 操作者必须在其`ClusterServiceVersion` (CSV)对象的`relatedImages`参数中列出操作者需要的所有容器图像。运算符使用摘要(安全哈希算法)而不是标签来引用指定的图像。

## APIcast Gateway 的新自定义指标政策

APIcast Gateway 的新[自定义指标政策](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#custom-metrics)允许您添加点击量以外的指标。在从后端 API 收到响应后，您可以使用该策略在上游响应代码或头上添加指标。

用例包括从后端跟踪 HTTP 2xx 响应或 HTTP 4xx 或 5xx 响应。在您只想跟踪成功响应代码的情况下，您可以将 HTTP 2xx 自定义指标用于应用程序计划费率限制或定价规则。您可以使用 4xx 或 5xx 自定义指标，在 3scale 管理门户的**分析**部分报告来自上游 API 的多个错误代码。另一个常见的用例是基于后端 API 发送的`content-length`头响应的定制指标。您可以将此指标用于应用程序计划，以跟踪有效负载大小，而不是基于速率限制或计费的点击量。

自定义指标基于后端 API 的响应，因此在实施新策略时要考虑以下因素:

*   在使用自定义指标之前，必须使用管理门户或管理 API 创建指标定义。
*   如果在请求被发送到上游 API 之前进行身份验证，您必须再次调用后端 API 管理器来报告新的指标。
*   自定义指标策略不适用于[3 规模批处理](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html/administering_the_api_gateway/apicast_policies#batcher)策略。

## 观看视频

观看使用新的 APIcast 自定义指标策略创建自定义指标的快速实时演示。

[https://www.youtube.com/embed/ESOjOWpNxYw?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/ESOjOWpNxYw?autoplay=0&start=0&rel=0)

## 3scale Management API 2.9 中的更多新功能

3scale Management API 2.9 版本中的其他新功能包括新的[上游互助 TLS 策略](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/administering_the_api_gateway/index#upstream-mtls)，新的[计费货币配置](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/admin_portal_guide/index#yaml_configuration_for_currencies)，以及 [API 后端的分析数据](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/admin_portal_guide/index#checking_analytics_for_backends)。其他[小功能更新](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html-single/release_notes_for_red_hat_3scale_api_management_2.9_on-premises/index#minor_features)见发布文档。

3scale Management API 2.9 版本还新增了 3scale Operator，它为 Prometheus 和 Grafana 带来了用于备份和恢复、自定义资源以及计量和监控资源的新功能。3scale Management API 2.9 版支持 ActiveDocs 上的 OpenAPI 3.0 规范。

## 每日伦敦:立即观看

APIDays 是针对 API 和可编程经济的领先行业技术和商业系列会议。2020 年 10 月 27 日和 28 日，红帽赞助了 APIDays LIVE London:通过 API 实现嵌入式金融、银行和保险之路。这个虚拟活动邀请了来自零售和投资银行、保险和金融的技术和业务领导者，解释他们如何使用 API 来创造新的业务价值。

注册可以免费观看 APIDays LIVE London 的会议记录，包括今年的两场 Red Hat 会议:

*   **2020 年 10 月 27 日星期二，格林威治时间中午 12:30**:*为什么你的数字身份在后 COVID 时代至关重要*，作者 EMEA 高级顾问卢卡·法拉利。
*   **2020 年 10 月 28 日星期三上午 10:50 GMT**:*一种开放银行业务的云原生方法*，由 FSI 营销经理 Rafael Marins 主讲。

两场会议都被记录下来，可以在 APIDays 伦敦活动页面上注册观看。

*Last updated: October 20, 2022*