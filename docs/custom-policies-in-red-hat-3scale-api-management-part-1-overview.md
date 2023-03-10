# Red Hat 3scale API 管理中的自定义策略，第 1 部分:概述

> 原文：<https://developers.redhat.com/blog/2021/02/24/custom-policies-in-red-hat-3scale-api-management-part-1-overview>

像[Red Hat 3 scale API Management](https://developers.redhat.com/products/3scale/overview)这样的 API 管理平台提供了一个 API 网关，作为 API 请求和响应之间的反向代理。在这个阶段，大多数 API 管理平台优化请求-响应路径，避免引入复杂的处理和延迟。这种平台提供了最低限度的策略实施，如身份验证、授权和速率限制。然而，随着基于 API 的集成的激增，客户需要更好的功能。

策略框架是向 API 请求和响应生命周期添加新功能的关键。在本系列中，您将了解 Red Hat 3scale API 管理策略框架，以及如何使用它在 [APIcast API 网关](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.6/html/administering_the_api_gateway/index)中配置自定义策略。

## 通过 3 级 API 管理实施策略

[APIcast](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html/administering_the_api_gateway/index) 是 3scale API 管理的默认数据平面网关，也是 API 请求和响应的策略执行点。其核心功能是实施速率限制、报告方法和指标，并使用为 3scale API manager 中定义的每个 API 指定的映射路径和安全性。

APIcast 建立在 [NGINX](https://www.nginx.com/) 之上。这是一个使用 [OpenResty](https://openresty.org/en/resources.html) 框架的反向代理的定制实现，模块用 [Lua](https://www.lua.org/docs.html) 编写。大多数 NGINX 功能是使用模块实现的，这些模块由配置文件中指定的指令控制。

APIcast 强制执行在 3scale API 管理器中设置的 API 配置规则。它通过连接到 API 管理器公开的服务 API 来验证新请求。它还允许访问后端 API 并报告使用情况。图 1 展示了 3scale API Management 的 API 请求和响应流中 APIcast 的高级视图。

[![](img/e26378959ed3d78fe1ddb226059a8591.png "3scale High level diagram")](/sites/default/files/blog/2020/06/3scale-High-level-diagram.png)

Figure 1: APIcast in the 3scale API Management API request and response flow.

## 默认的 APIcast 策略

默认 APIcast 策略解释标准配置并提供 API 网关功能。默认策略充当 API 到网关的入口点，并且必须为 3scale API Management 中配置的所有 API 启用。APIcast 策略确保使用 API 管理器中配置的规则处理 API 请求。该配置作为 JSON 规范提供给 APIcast 网关，APIcast 从 3scale API 管理门户下载该规范。

每个 HTTP 请求都经过一系列的[阶段](http://nginx.org/en/docs/dev/development_guide.html#http_phases)。在每个阶段，对请求执行不同类型的处理。特定于模块的处理程序可以在大多数阶段中注册，许多标准 NGINX 模块注册它们的阶段处理程序，以便在请求处理的特定阶段调用它们。阶段是连续处理的，一旦请求到达阶段，就会调用阶段处理程序。

为了定制请求处理，我们可以在适当的阶段注册额外的模块。3scale API 管理提供标准策略，这些策略预构建为 NGINX 模块，可以插入到每个服务的请求中。

## 自定义 APIcast 策略

除了默认策略之外，3scale API 管理还提供自定义策略，可针对每个 API 进行配置。通过使用模块和配置，这些策略提供了处理 API 请求和响应的定制特性。使用定制模块使得 APIcast 网关具有高度的可定制性。无需修改 API 网关代码或编写任何额外的代码，就可以按需添加自定义处理和功能。

**注意**:参见 3sale API 管理文档中关于 APIcast 策略的[章节，了解可直接使用 APIcast 配置的所有标准策略列表。](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html/administering_the_api_gateway/apicast_policies#standard-policies)

## 策略链

策略必须按照执行优先级的顺序放置，这种放置称为*策略链*。策略链影响任何策略组合的默认行为。默认的 APIcast 策略应该是策略链的一部分。如果自定义策略需要在 APIcast 策略之前进行评估，则必须将其放在链中该策略之前。图 2 显示了策略链中定义的策略顺序的示例。

[![A diagram of the custom policy chain.](img/275cb311bf48bf5ce8e55056c519f4a1.png "Policy Chain")](/sites/default/files/blog/2021/02/Policy-Chain.png)

Figure 2: A custom URL-rewriting policy in the policy chain.

例如，假设使用 URL 重写策略将 URL 路径从 */A/B* 更改为 */B* 。将 URL 重写策略放在 APIcast 策略之前，可以确保在网关处理之前更改路径。后端规则、映射规则和指标都将使用 */B* URL 路径进行评估。

另一方面，如果自定义策略应该在 APIcast 策略的之后*进行评估，您可以颠倒顺序。例如，如果您希望为 */A/B* 评估映射规则，并在之后应用到 */B* 的 URL 重写，那么您应该将 URL 重写策略放在 APIcast 策略之后。*

## 配置自定义策略

向 API 添加新策略有两种方式。一种选择是在 3scale API 管理管理门户中为每个受管 API 使用策略部分。所有可用的策略都可以添加。如果您更喜欢使用 Admin API，那么您可以将策略作为 JSON 规范提供，您可以使用提供的 REST API 将它上传到服务配置。

您还可以使用 [3scale Toolbox](https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html/operating_3scale/the-threescale-toolbox) 将一组策略作为服务配置的一部分从一个运行环境复制到另一个运行环境。要验证应用于特定 API 的策略集，您可以从管理门户或使用提供的 REST API 下载当前配置和配置历史。

查看以下视频，了解 3scale API 管理中的策略配置演示。

[https://www.youtube.com/embed/8R5FvYE9JC0?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/8R5FvYE9JC0?autoplay=0&start=0&rel=0)

## 结论

3 扩展 API 管理为配置自定义 API 策略提供了多个选项。本文介绍了 3scale API 管理策略框架、自定义策略和策略链。我还简单介绍了如何在 3scale API 管理中配置和查看策略。本系列的后续文章将研究可用的策略，我将介绍开发人员工具集，您可以用它来创建自己的定制策略。同时，您可以通过注册一个[免费的 3scale API 管理帐户](https://www.3scale.net/signup)来探索 3scale API 管理。

*Last updated: May 6, 2021*