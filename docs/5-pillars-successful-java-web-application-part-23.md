# 成功的 Java Web 应用程序的 5 大支柱(第 2/3 部分)

> 原文：<https://developers.redhat.com/blog/2017/11/07/5-pillars-successful-java-web-application-part-23>

在这一系列的帖子中，我们将详细介绍我们在 Java One San Francisco 2017 上的演讲:“ [一个成功的 Java Web 应用的 5 大支柱](https://speakerdeck.com/ederign/5-pillars-of-a-successful-java-web-application-1) ”，在这里我们分享了多年来为 Drools 和 jBPM 平台构建工作台和 Web 工具的累积经验。如果您没有阅读第一篇文章，请花点时间阅读第一篇文章[。](https://developers.redhat.com/blog/2017/11/06/5-pillars-successful-java-web-application-part-13/)

## 第二支柱:全栈开发者

每一个成功的 web 应用程序的第二个支柱都与开发人员的技能有关:我们应该拥有完整的堆栈。你的公司可能仍然区分后端和前端开发人员，但这个界限将逐渐消失，因为最终，我们是开发人员，开发人员应该解决问题。问题出在服务器上还是浏览器上并不重要，因为它们只是解决问题的媒介。

在这种全栈环境中，最有效的工作方式是对后端和前端使用相同的编程模型。在我们的团队中，我们采用了 Java EE 编程模型(当然我们将在即将到来的 EE4J 中扮演重要角色)，但是我们如何在浏览器中共享相同的 Java EE 编程模型呢？

为此，我们使用了 [Errai](http://erraiframework.org/) 项目。利用 GWT 编译器，Errai 使您能够在客户机上重用现有的 Java EE (Eclipse EE)代码。使用 Errai，您可以在客户端代码中进行依赖注入，在客户端观察和触发 CDI 事件，并在客户端和服务器之间交换事件。

在我们的应用程序的所有层中拥有相同的编程模型，使它发展得更快更安全，尤其是减少了后端和前端编程模型之间的上下文切换。在这里 了解 Errai 的 Java EE 特性 [。](https://github.com/errai/errai-tutorial)

## 第三大支柱:UX 一体化

成功的网络应用的下一个支柱是促进与 UX 团队的整合。你的 UX 团队有足够的知识来构建易用且视觉上吸引人的用户界面。这不仅仅是一份工程类的工作:这需要不同的技能组合，它们必须协同工作才能成功。

混合 HTML/CSS 和控制逻辑语言是一个错误。在维护 JSP 页面时，我们经历了惨痛的教训。

不幸的是，今天许多 JS 框架正在走向同样的道路:

<h1>{ { title } }</h1>

< h2 >我喜欢的英雄是:{{myHero}} < /h2 >

< p >英雄:< /p >

<ul>

<李*ngFor= "让英雄出英雄">

{{ hero }}

</李>

</ul>

一个 UX 专家怎么能研究这种代码呢？让程序员“翻译 HTML/CSS”到这个框架的细节中有什么限制？我们的行业不断迫使 UX 理解特定于框架的代码并与之交互。

第三个支柱是，你的 web 应用程序应该尊重 HTML 和 CSS，并尽可能保持干净。这是 UX 和工程师无缝结合的唯一途径。但是如何实现这一点呢？

[Errai](http://docs.jboss.org/errai/latest/errai/reference/html_single/#sid-51806600) 提供了一个纯粹的基于 HTML/CSS 模板的框架。使用注释处理器，我们可以透明地将 HTML 标签绑定到 java 代码中的 DOM 元素，而不会对 HTML/CSS 结构带来任何改变。使用 Errai UI，我们不会将业务逻辑与 HTML/CSS 混合搭配。

这对 Drools 和 jBPM 团队很有帮助，因为它允许 UX 和工程团队之间无声的集成。因此，web 应用程序的第三个支柱是与 UX 团队紧密合作，而有效做到这一点的唯一方法是尽可能保持 HTML 和 CSS 的整洁。

这是三篇关于成功网络应用五大支柱的文章中的第二篇。敬请期待下一期。

我要感谢 Max Barkley 和 Alexandre Porcelli 在本文发表前审阅了本文，为最终文本做出了贡献，并提供了很好的反馈。]

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/docker-cheatsheet/) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: November 6, 2017*