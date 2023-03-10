# 借助 Red Hat 3scale API 管理，添加 API 网关策略变得更加容易

> 原文：<https://developers.redhat.com/blog/2018/05/30/3scale-api-gateway-policies>

随着 2018 年 6 月发布的 Red Hat 3scale API Management 2.2，将 API 网关策略添加到您的 API 管理层比以往任何时候都更容易。

## **什么是政策？**

Red Hat 3scale API 管理提供了修改 API 网关行为的功能单元，无需实施代码。这些管理组件在 3 个级别中被称为**策略**。捆绑策略的配置可从 API Manager 门户获得，您可以在其中定义 API 集成的行为。

策略的执行顺序称为“策略链”,可根据策略在链中的位置进行配置，以引入不同的行为。添加自定义头、执行 URL 重写、启用 CORS 和可配置缓存是作为策略实现的一些最常见的 API 网关功能。

![](img/1daa0455a4d57f25d2b24d18ae6ef9e0.png)

【3scale 提供了各种现成的策略，但您并不限于此。借助 3scale，您可以完全访问网关策略框架，以编写自定义代码，在内置于 API 管理的基本 APIcast 策略之上实现新的 API 网关功能。

![3scale standard policies](img/a05e4809b6270a582750e55214949958.png)

## **3 规模标准政策**

Red Hat 3scale API 管理提供了许多标准策略:

*   **CORS: 跨源资源共享(CORS)请求处理策略—允许您控制 CORS 行为**
*   **URL 重写:** 允许您使用 OpenResty web platform sub 和 gsub 操作修改请求的路径。
*   **Echo:** 将传入的请求打印回客户机，并附带一个可选的 HTTP 状态代码。
*   **添加标头:** 标头策略允许您修改现有标头或定义附加标头，以添加到传入请求或响应中或从中删除。
*   **上游:** 允许您使用正则表达式解析主机请求头，并用新的 URL 替换请求头 URL。
*   **SOAP:**SOAP 策略将 HTTP 请求的 SOAP action 或 Content-Type 头中提供的 SOAP action URIs 与策略中指定的映射规则相匹配。
*   **离线操作通信:** 认证缓存策略缓存对 API 网关的认证调用。您可以通过选择操作模式来配置缓存的操作方式。

## **好处**

3 scale API 管理的模块化策略的优势有:

*   通过数据而不是代码进行配置
*   为请求周期的任何阶段添加具有新策略的网关逻辑的能力
*   更好的扩展性
*   改进的可维护性
*   社区贡献的杠杆作用

因此，看看下面的视频。它概述了如何在 3scale API 管理中启用标准策略。

[https://www.youtube.com/embed/836_E_2zZfg?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/836_E_2zZfg?autoplay=0&start=0&rel=0)

*Last updated: September 3, 2019*