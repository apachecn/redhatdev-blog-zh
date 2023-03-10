# 用 Jaeger 和 Eclipse Che 跟踪 Kubernetes 应用程序

> 原文：<https://developers.redhat.com/blog/2019/11/14/tracing-kubernetes-applications-with-jaeger-and-eclipse-che>

开发分布式应用程序是复杂的。您可以等到在测试或试运行服务器上启动应用程序时，或者如果幸运的话，在生产环境中启动应用程序时，再监视性能问题，但是为什么不在开发过程中跟踪性能呢？这允许您在将变更推广到测试或生产环境之前发现改进的机会。本文演示了两个工具如何协同工作来将性能监控集成到您的开发环境中: [Eclipse Che](https://www.eclipse.org/che/) 和 [Jaeger](https://www.jaegertracing.io/) 。

据 Eclipse Che 网站报道:

*“Che 将您的 Kubernetes 应用程序引入您的开发环境，并提供了一个浏览器内 IDE，允许您编码、构建、测试和运行应用程序，就像它们在任何机器上运行一样。”*

在本文中，我们展示了将 Jaeger 添加到 Eclipse Che 开发工作区是多么简单，并观察了 Kubernetes 应用程序的执行情况。我们将使用 [che.openshift.io](http://che.openshift.io/) 作为托管环境，尽管如果你愿意，你可以[建立一个本地 che 服务器](https://www.eclipse.org/che/docs/che-7/running-che-locally/)。

## 创建工作空间

Che 7 引入了以 YAML 格式定义开发工作区的功能，称为 devfile。示例 devfiles 可以在[红帽开发者 GitHub](https://github.com/redhat-developer/devfile) 中找到。

在这篇文章中，我们使用了一个修改版的 [Spring Boot 入门开发文件](https://github.com/redhat-developer/devfile/blob/master/getting-started/spring-boot/devfile.yaml)，它将 Jaeger 一体机后端添加到了工作区。主要变化是在顶层节点`commands`之前添加以下部分:

```
-  
    type: dockerimage  
    alias: tracing  
    image: jaegertracing/all-in-one:latest  
    env:  
      - name: MEMORY_MAX_TRACES  
        value: "5000"  
      - name: COLLECTOR_ZIPKIN_HTTP_PORT  
        value: "9411"  
    memoryLimit: 128Mi  
    endpoints:  
      - name: 'tracing-ui'  
        port: 16686  
      - name: 'collector-grpc'  
        port: 14250  
        attributes:  
           public: 'false'  
      - name: 'collector-http'  
        port: 14268  
        attributes:  
           public: 'false'  
      - name: 'collector-zipkin'  
        port: 9411  
        attributes:  
           public: 'false'  
      - name: 'agent-config'  
        port: 5778  
        attributes:  
           public: 'false'  
      - name: '6831/udp'  
        port: 6831  
        attributes:  
           public: 'false'  
      - name: '6832/udp'  
        port: 6832  
        attributes:  
           public: 'false'  
    volumes:  
      - name: tmp  
        containerPath: /tmp 
```

devfile 的完全修改版本可以在[这里](https://gist.github.com/objectiser/667615926a40d6cd8eb675859ddee1a1)找到，使用 [che.openshift.io](http://che.openshift.io) 需要额外的内存限制变化。

要在 [che.openshift.io](http://che.openshift.io) 上启动工作区，将浏览器指向[这个 url](http://che.openshift.io/f?url=https://gist.githubusercontent.com/objectiser/667615926a40d6cd8eb675859ddee1a1/raw/06d25e2026d7a689dd8b38a343eec9a9cc431cde/che-spring-boot-devfile.yaml) 。由于某些版本的 Firefox 存在问题，建议使用 Chrome。

## 添加 OpenTracing 检测

当工作区最初打开时，应用程序没有 OpenTracing 工具，如图 1 所示:

[![Figure 1: The initial workspace.](img/d337a6aaf358602db0cab60c92c46303.png "jaeger-che1")](/sites/default/files/blog/2019/11/jaeger-che1.png)

图 1:初始工作区。">

通过包含对`opentracing-spring-jaeger-cloud-starter`的依赖(如图 2 中更新的`pom.xml`文件所示)，以及将`spring-boot-starter-parent`版本更新为`2.2.0.RELEASE`(这是 OpenTracing instrumentation 所需要的)，可以隐式地添加 OpenTracing instrumentation:

[![OpenTracing](img/c2845f599f9b39e96c5e3b3a7470d4ae.png "jaeger-che2")](/sites/default/files/blog/2019/11/jaeger-che2.png)

图 2:添加 OpenTracing 工具。">

这种依赖性自动检测入站和出站 HTTP 请求。它还引导 Jaeger tracer 向 Jaeger 后端(包含在工作区中)报告跟踪数据。默认的 tracer 配置将通过 UDP 向 Jaeger 代理报告数据，尽管应用程序可以配置为通过 [HTTP 直接向收集器](https://github.com/opentracing-contrib/java-spring-jaeger#configuration)报告数据。

最后一步是添加一个在跟踪数据中定义服务名称的属性。这个目标是通过创建`src/main/resources`文件夹，然后用图 3 所示的内容创建文件`application.properties`来实现的:

[![](img/bfcac4f78240ad5797102e7e54ef5369.png "jaeger-che3")](/sites/default/files/blog/2019/11/jaeger-che3.png)

图 3:在跟踪数据中定义服务名。">

## 跟踪正在运行的应用程序

工作空间的右侧是一个立方体符号。选中时，此图标会展开一个树。在**用户运行时/工具**树节点下是一个名为**运行 webapp** 的任务。选择此选项将运行 Spring Boot 应用程序。启动时，出现一个窗口，带有按钮**打开链接**，如图 4 所示。按此按钮启动应用程序的浏览器。

[![Figure 4: Opening a browser for the application.](img/136d89ac04d97fe769f84a1c08aeb3ef.png "jaeger-che4")](/sites/default/files/blog/2019/11/jaeger-che4.png)

图 4:打开应用程序的浏览器。">

在同一个树中，选择**用户运行时/跟踪**选项**跟踪-ui** ，这将在一个单独的浏览器选项卡中启动 Jaeger UI，如图 5 所示:

[![](img/f58119f42bfb2e5867e270532d031554.png "jaeger-che5")](/sites/default/files/blog/2019/11/jaeger-che5.png)

图 5:启动一个新的跟踪浏览器选项卡。">

按几次浏览器顶部的刷新按钮，在控制台窗口中看到文本`Span reported`，如图 6 底部所示:

[![](img/903b0d2bed423048e4c27b2c8bc930a7.png "jaeger-che6")](/sites/default/files/blog/2019/11/jaeger-che6.png)

图 6:运行应用程序，控制台日志显示报告的跨度。">

切换到 **Jaeger UI** 选项卡，查看应用程序报告的结果轨迹，如图 7 和图 8 所示:

[![Figure 7: The completed traces.](img/e9039a35eb897823724bf27ee776a1ab.png "jaeger-che7")](/sites/default/files/blog/2019/11/jaeger-che7.png)

图 7:完成的轨迹。">

[![](img/91d7afa7f2c87663141891aa174e0528.png "jaeger-che8")](/sites/default/files/blog/2019/11/jaeger-che8.png)

图 8:第一个完成的跟踪的细节。">

## 摘要

本文展示了如何将带有 Jaeger 的 OpenTracing 轻松引入 Eclipse Che 工作区，以便您可以在开发期间从应用程序获取跟踪信息。

这个特殊的例子很简单，只捕获单个服务的跟踪。Che 提供的好处是使完整的应用程序(多个服务)能够在同一个工作空间中使用，从而产生更有趣的跟踪，并帮助开发人员了解他们开发的服务在完整应用程序环境中的性能。

*Last updated: May 27, 2022*