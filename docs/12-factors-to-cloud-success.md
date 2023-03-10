# 云成功的 12 个因素

> 原文：<https://developers.redhat.com/blog/2017/06/22/12-factors-to-cloud-success>

嗨，开发者们！您关心使用最佳实践将您的应用程序应用到云中吗？如果是这样，那么你应该使用[12 因素应用](https://12factor.net/) ，这是一种构建软件即服务的方法。今天我想谈谈 12 因素应用程序，这是我上个月在红帽峰会上向一个小组展示的。

每个将应用程序迁移到云的开发人员都将面临一个不同于他们所习惯的环境，他们的数据中心或自己的场所，这就是为什么他们应该关注 12 因素方法。这种 12 步方法是由 Heroku 创建的，Heroku 是一家云提供商，他们找到了一种通用的解决方案来解决他们的大多数客户正在经历的问题，并决定将这些解决方案作为一种方法来发布。这 12 个因素旨在解决与云中运行的应用程序相关的问题。如果从我的演讲中有一个关键的收获的话，那不是让观众记住这 12 个因素，而是理解为什么每一个都很重要。

1.  **代码库**——使用版本控制，一个代码库在许多部署的版本控制中被跟踪。
2.  **依赖关系**——使用包管理器，不要在代码库中提交依赖关系。
3.  将配置存储在环境变量中，如果你不得不重新打包你的应用程序，你这样做是错误的。
4.  **后台服务**——一个 [部署](https://12factor.net/codebase) 的十二要素 app 应该能够将一个本地 MySQL 数据库换成一个由第三方管理的数据库(比如 [亚马逊 RDS](http://aws.amazon.com/rds/) )，而不需要对 app 的代码做任何改动。
5.  **构建、发布、运行**——十二要素应用在构建、发布和运行阶段之间使用了严格的分离。每个版本都应该有一个唯一的版本 ID，并且版本应该允许回滚。
6.  **进程**——将 app 作为一个或多个无状态进程执行，十二要素进程无状态 [无共享](http://en.wikipedia.org/wiki/Shared_nothing_architecture) 。
7.  **端口绑定**——通过端口绑定导出服务，十二因子 app 是完全独立的。
8.  **并发**——通过流程模型向外扩展。每个流程都应该单独扩展，使用因子 6(无状态)，很容易扩展服务。
9.  **一次性**——最大限度地提高快速启动和平稳关闭的稳健性，我们可以通过容器实现这一点。
10.  **开发/生产对等**——尽可能保持开发、试运行和生产的相似性，十二因素应用程序旨在通过保持开发和生产之间的较小差距来实现 [持续部署](http://www.avc.com/a_vc/2011/02/continuous-deployment.html) 。
11.  **日志**——将日志视为事件流，一个 12 因素的应用从不关心其输出流的路由或存储。
12.  **管理进程** -作为一次性进程运行管理任务。

【T12 因素 App 方法论是技术和语言不可知的，但满足于 [容器](https://developers.redhat.com/containers/)[微服务](https://developers.redhat.com/microservices/) ，以及 CI/CD 管道，重点是[devo PS](https://developers.redhat.com/devops/)。 您可以在[【https://12factor.net/】](https://12factor.net/)访问 12 因素 App 了解更多信息。

* * *

**我在 2017 红帽峰会上提出了[云成功的 12 个因素:](https://www.youtube.com/watch?v=tU01aPrb5K0&amp&list=PLEGSLwUsxfEjolewXub1rSgvILhc2OYQS&index=31)**

[https://www.youtube.com/embed/tU01aPrb5K0](https://www.youtube.com/embed/tU01aPrb5K0)

*Last updated: October 18, 2018*