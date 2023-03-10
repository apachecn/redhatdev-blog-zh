# 作为产品的 API:从 API 中获取价值

> 原文：<https://developers.redhat.com/blog/2019/12/02/apis-as-a-product-get-the-value-out-of-your-apis>

API 继续蔓延，正如来自 [ProgrammableWeb](https://www.programmableweb.com/news/apis-show-faster-growth-rate-2019-previous-years/research/2019/07/17) 的 2019 年报告所示，该报告显示比去年的增长率增长了 30%。越来越多的法规强制使用 API 来开放公司和促进创新。想想[支付服务指令第二版(PSD2)](https://www.gemalto.com/financial/ebanking/psd2) ，[开放银行](https://www.redhat.com/en/resources/open-banking-technology-overview?source=searchresultlisting)，以及公共部门发布 0pen 数据 API。有了如此丰富的 API，从 API 中获取价值并在日益激烈的竞争中脱颖而出变得越来越重要。是时候将 API 作为产品来设计和管理了。

## 改变你的组织

将一个 API 设计成一个产品意味着 API 的设计和管理方式的大量改变。这意味着将你的思维模式从“包装现有服务”转变为“满足客户需求”。对于负责 API 的团队来说，这种心态意味着负责产品，因此:

*   关注客户需求而不是现有资产。
*   [持续管理 APIs】而不是在项目的整个生命周期中。](https://developers.redhat.com/blog/2019/02/25/full-api-lifecycle-management-a-primer/)
*   从有限资源转向弹性计算。
*   从一个专注于 API 的核心团队发展成为拥有多种能力的跨职能产品团队。

简而言之，将 API 设计成产品意味着:

*   专为解决客户问题而设计。
*   设置 API 交付价值的共享指标。
*   有一个反馈回路来知道要改进什么。

## 改变你的设计过程

![Use API design thinking to design APIs as a product.](img/49a4ab6f6f523a45afe13a3783fdd78e.png)

将 API 设计成产品也意味着改变我们制作 API 的方式(通过采用 *API 设计思维*):

1.  应用于 API 的设计思想的第一步是关注客户。透过他们的眼睛，他们的痛苦是什么？*移情*意味着与你的潜在新客户会面并倾听他们的心声。你会发现他们的理解，他们是如何组织的，他们使用哪种技术生态系统等等。
2.  然后，你将综合你的发现来*定义*他们试图解决的问题，并确定[要做的工作](https://jtbd.info/)。想要执行交易来购买宠物的客户和想要同步他们的宠物库存的客户将导致两个非常不同的 Petstore APIs。
3.  一旦定义好了，你就可以[通过一个叫做 API 构思](https://developers.redhat.com/blog/2018/04/11/api-journey-idea-deployment-agile-part1/)的过程来培养新的想法。在这一步中，您将研究不同的 API 设计，看看是否有一个可行的解决方案。
4.  在 *API 原型制作*阶段，你[基于有意义的例子](https://developers.redhat.com/blog/2018/04/19/api-journey-idea-deployment-agile-way-part2/)模仿你的 API，最终得到一个工作的 API。其他团队成员可以对您的设计尝试给出反馈。
5.  倒数第二步是*从早期采用者那里获得对最终设计的反馈*,从而提炼出对 API 的业务期望。
6.  最后，您*实现*实际的 API。

## 开源社区帮助你将 API 设计成一个产品

到目前为止，我们只谈到了组织变革。专用工具和开源社区可以帮助您成功地将 API 作为产品进行设计和管理:

*   [Microcks](http://microcks.github.io/) 是开源的 Kubernetes-用于 API 模拟和测试的原生工具，在*构思*和*原型*阶段帮助你。
*   Apicurio 帮助你以协作的方式设计更好的 API 契约。架构师、产品负责人、设计人员和开发人员可以一起进行 API 设计。
*   [Red Hat 3scale API 管理](https://developers.redhat.com/products/3scale/overview)通过推广 API 设计、收集反馈以及将 API 作为产品进行实际管理，简化了 API 作为产品的设计和管理。

[在下一篇文章](https://developers.redhat.com/blog/2019/12/03/apis-as-a-product-get-started-in-no-time/)中，我们将看看 3scale 的最新版本如何帮助您将 API 作为产品来构建。

*Last updated: July 1, 2020*