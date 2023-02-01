# 第 3 部分:运行中的反应系统

> 原文：<https://developers.redhat.com/coderland/reactive/reactive-system-in-action>

至此，我们已经介绍了代码是如何工作的。我们有一个真正的反应式系统，它使用一组微服务来完成工作。现在是实际部署和运行应用程序的时候了。

## **部署和运行 Reactica 代码**

现在我们有了一个真正的反应式系统，它使用一组微服务来完成工作，所以是时候实际部署和运行应用程序了。幸运的是，该代码的作者(Clement Escoffier、James Falkner、Thomas Qvarnströ和 Rodney Russ，他们中没有一个人值得高度赞扬)创建了 shell 脚本来简化代码的部署和运行。这些脚本做了大量工作来创建一个 Kubernetes 集群并将应用程序部署到其中。这是好消息。不太好的消息是，因为这些脚本要做很多工作，所以需要一段时间。所以，拿一杯咖啡，一本像样的书，或者一些有益健康的小吃，坐下来休息几分钟。

## **先决条件**

在部署和运行 Reactica 之前，您需要在您的路径上安装`minishift` 1.33.0 或更高版本。在我们的测试中，我们使用了从 https://github.com/minishift/minishift/releases 下载的 Mac 版`minishift`的 1.34.0 版本。你也可以从[https://developers.redhat.com/products/cdk/overview](https://developers.redhat.com/products/cdk/overview)下载红帽容器开发者工具包来获得`minishift`。如果出于某种原因，你还没有注册红帽开发者计划，你需要注册。(说真的，你怎么还没报名呢？它是免费的。)

除了`minishift`，脚本还需要`curl`和 [Maven](http://maven.apache.org/) 版本 3.5.3 或更高版本。

## 部署基础设施

如果你还没有，从[https://github.com/reactica/rhte-demo](https://github.com/reactica/rhte-demo)克隆或派生 Reactica 代码。现在运行以下命令:

*   `minishift profile set rhte-vertx-demo`

*   `minishift stop`

*   `minishift delete`

*   `setup/deploy.sh `

将轮廓设置为`rhte-vertx-demo`。接下来，运行`minishift stop`和`minishift delete`以确保我们都从同一个地方开始。如果您收到一条消息，询问您是否确定要删除该简档，请说“是”。`deploy.sh`脚本在您的机器上创建并配置一个新的集群，所以这显然需要一段时间。现在就去拿那杯咖啡吧。在你的作者的机器上，花了大约 18 分钟。请注意，该脚本创建了一个使用 8gb RAM 和 3 个 vCPUs 的集群。在你开始脚本之前，你需要尽可能的关闭所有的东西。

如果`deploy.sh`运行成功，它应该会成功，我们有一个正在运行的 OpenShift 集群，其中运行着红帽 AMQ 和红帽数据网格。该脚本启动数据网格和 AMQ，在继续之前等待每个的状态为`ready`。*完全披露:*根据你机器的处理能力，你可能会得到一个超时错误。如果这是真的，试着再次运行`setup/deploy.sh`。即使脚本超时并停止运行，系统也会继续在后台部署数据网格或 AMQ。当你再次运行脚本时，它会跳过所有已经运行的程序。不过，第一次几乎总是魅力所在。我们保证。

## 配置杯垫

接下来，创建一个 Kubernetes `ConfigMap`来设置过山车的参数。配置设置包括新的`User`多久排队一次，多少人可以同时乘坐一辆`Ride`，以及一辆`Ride`持续多长时间。

切换到`setup`目录。它包含文件`application.yaml`。您可能想要修改的三个属性是:`user-simulator/period-in-seconds`定义新的`User`应该多久生成一次，`ride-simulator/duration-in-seconds`设置过山车每次往返的持续时间，`ride-simulator/users-per-ride`定义多少人可以同时乘坐`Ride`。

这是 YAML 的原始文件:

```
---
user-simulator:
  period-in-seconds: 5
  jitter-in-seconds: 2
  enabled: false

ride-simulator:
  duration-in-seconds: 30
  users-per-ride: 5
  jitter-in-seconds: 5
  enabled: true
```

运行脚本`setup/create-application-config.sh`会基于 YAML 文件中的值创建一个名为`reactica-config`的 Kubernetes 配置图。如果您想再次更改配置，您需要编辑配置图(`oc edit configmap reactica-config`)。这将使你进入你的系统编辑器，不管它可能是什么(`vi`、`nano`、`emacs`等等)。).保存您的更改，系统将使用新设置。

我们鼓励你尝试不同的价值观，看看它是如何影响广告牌的。

## 展开杯垫

最后一步是在集群中部署应用程序。运行`setup/deploy-application.sh`来实现。这启动了一个 Maven 构建和其他脚本，安装运行在红帽 AMQ 和红帽数据网格之上的反应式微服务。这也需要几分钟，但是最后几行输出应该是这样的:

```
✔️  Pod event-generator is Running
✔️  Pod event-generator is ready
✔️  Pod event-store is Running
✔️  Pod event-store is ready
✔️  Pod current-line-updater is Running
✔️  Pod current-line-updater is ready
✔️  Pod queue-length-calculator is Running
✔️  Pod queue-length-calculator is ready
✔️  Pod billboard is Running
✔️  Pod billboard is ready

```

“`Running`”和“`ready`”是我们要找的，所以一切都好。要访问该应用程序，您需要知道它的地址，以便可以在浏览器中打开它。键入`oc get routes`查看进入集群的路由:

```
oc get routes
NAME                      HOST/PORT                                                          PATH SERVICES         PORT                TERMINATION   WILDCARD
billboard                 billboard-reactive-demo.192.168.64.37.nip.io                       billboard             8080                None
console                   console-reactive-demo.192.168.64.37.nip.io                         eventstream-amq-jolokia <all>             None
current-line-updater      current-line-updater-reactive-demo.192.168.64.37.nip.io            current-line-updater  8080                None
event-generator           event-generator-reactive-demo.192.168.64.37.nip.io                 event-generator       8080                None
event-store               event-store-reactive-demo.192.168.64.37.nip.io                     event-store           8080                None
eventstore-dg             eventstore-dg-reactive-demo.192.168.64.37.nip.io                   eventstore-dg         <all>               None
queue-length-calculator   queue-length-calculator-reactive-demo.192.168.64.37.nip.io         queue-length-calculator   8080 None
```

我们正在寻找`billboard`垂直的 URL。从这个输出我们可以看到它的地址是`billboard-reactive-demo.192.168.64.37.nip.io`。(您机器上的地址几乎肯定会不同。)将其剪切并粘贴到您最喜欢的浏览器中，您应该会看到如下内容:

![Reactica web UI](img/271b1e6fea67adc9b98913ad47c7a612.png "Reactica web UI")

这是游乐设备状态页面；它告诉我们有多少人在排队以及等待时间。从截图中可以看到，显示中有 10 名骑手。底部的三个(绿色)已经完成了乘坐，中间的四个(黄色)正在乘坐，顶部的三个(蓝色)正在排队等候。

**注意:**绿色是随机选择的，尽管许多用户在乘车时确实会变绿。

但是，在进入该面板之前，您需要进入游乐设备管理选项卡，并点击开始用户生成:

![Reactica admin UI](img/c2322ec58f2110b383d67c1117bf08d9.png "Reactica admin UI")

这将告诉`event-generator`vertical 开始生成新的`User`。现在切换回行驶状态面板。几秒钟后，您应该会看到一些人在排队。

## 管理游乐设备

你可以从管理面板做更多的事情。单击停止用户生成会阻止`event-generator`垂直生成更多的`USER_IN_QUEUE`事件。单击刷新队列会删除队列中的所有来宾。

请注意，`billboard`组件仍然会响应由`current-line`vertical 生成的事件。刷新队列时，显示会被清除。然而，有可能目前有客人在`ON_RIDE`州。当旅程结束时，`current-line`将他们的状态更改为`RIDE_COMPLETED`，而`billboard`vertical 将这些客人添加到空桌上。返回游乐设备管理面板，再次点击刷新队列，删除这些用户并清空表格。从那时起，在您再次单击“启动用户生成”之前，表中不会添加任何内容。

## 使用日志

如果您想深入了解到底发生了什么，每个组件都有一个日志。要访问它们，首先用`oc get pods`命令获取 pod 的名称:

```
NAME                              READY STATUS      RESTARTS AGE
billboard-1-s6wrg                 1/1   Running     0        1h
billboard-s2i-1-build             0/1   Completed   0        5h
current-line-updater-1-dvq2t      1/1   Running     0        1h
current-line-updater-s2i-1-build  0/1   Completed   0        5h
event-generator-1-h5krz           1/1   Running     0        1h
event-generator-s2i-1-build       0/1   Completed   0        5h
event-store-1-p455g               1/1   Running     0        1h
event-store-s2i-1-build           0/1   Completed   0        5h
eventstore-dg-1-tqcw6             1/1   Running     0        1h
eventstream-amq-1-24fkq           1/1   Running     0        1h
queue-length-calculator-1-6r5z6   1/1   Running     0        1h
queue-length-calculator-s2i-1-build   0/1 Completed 0        5h
```

要查看生成事件的流程，请键入`oc logs event-generator-1-h5krz`(或者您的`event-generator` pod 的名称)。以下是日志中的一些典型行:

```
17:45:47.215 [vert.x-eventloop-thread-1] INFO  com.redhat.coderland.reactica.UserSimulatorVerticle - Creating user Spangle Runner
17:45:51.224 [vert.x-eventloop-thread-1] INFO  com.redhat.coderland.reactica.UserSimulatorVerticle - Creating user Root Follower
17:45:51.433 [vert.x-eventloop-thread-0] INFO  com.redhat.coderland.reactica.RideSimulator - Ride 7b9e3738-56b8-4911-a83d-7be0537067c6 completed
17:45:51.451 [vert.x-eventloop-thread-0] INFO  com.redhat.coderland.reactica.RideSimulator - The users [Typhoon Chopper, Oasis Bat, Maze Arm, Tar Coyote, Fast Ape, Flint Mask, Shard Flasher, Magenta Braid] have completed their ride, bye bye!
17:45:51.452 [vert.x-eventloop-thread-0] INFO  com.redhat.coderland.reactica.RideSimulator - Onboarding ride {"uuid":"7877df24-3a76-44e6-88d1-9c88f7468ae5","state":"PLANNED"}
```

您可以从日志中的消息看到各种对象的生命周期，并看到哪个类生成了该事件。在这些消息中，几个`User`被创建，一个`Ride`完成，10 个`User`完成`Ride`(台风直升机，绿洲蝙蝠等。)，另一个`Ride`诞生了。如果刚刚记录了这些消息，您应该在队列中看到 Root Follower 和 Spangle Runner，并且应该看到上面列出的 10 个用户变成绿色，表示他们已经完成了乘坐。

你也可以在这里看到 Vert.x 的[多反应堆模式](https://vertx.io/docs/vertx-core/java/#_reactor_and_multi_reactor)的运行。Vert.x 在每个垂直方向上附加一个事件循环(物理 CPU 核心数量的 2 倍)。`RideSimulatorVerticle`正在一个事件循环(线程 0)上运行，而`UserSimulatorVerticle`正在另一个事件循环(线程 1)上运行。Vert.x 的架构允许它有效地利用底层系统的硬件功能。

## 摘要

这是 Reactica 过山车如何工作的完整旅程。尽管反应式系统本身很复杂，但每个微服务都相当简单。生产者生成的数据和消费者处理的数据会产生一个复杂的 web 用户界面，显示系统中发生的所有事情的结果。

要更进一步，请看一下这些资源:

*   要了解更多关于反应式编程，特别是 Vert.x 的知识，我们不能不高度推荐 Clement Escoffier 的优秀电子书《用 Java 构建反应式微服务:异步和基于事件的应用程序设计》,这是 Red Hat 开发者计划免费提供的。

*   你错过的反应式编程介绍

*   [反动宣言](https://www.reactivemanifesto.org)

*   [vert . x 网站上的文档和示例](https://vertx.io)

*   最后，如果你还没有看完本教程的第二部分,那就值得花时间看一遍。

如果你还不是红帽开发者计划的成员，今天就注册吧！您可以访问大量内容和红帽产品，包括红帽 AMQ 和红帽数据网格。

我们希望你喜欢 Coderland 的这个新功能。一如既往，请将您的意见、问题、建议和建议发送给我们。我们是 coderland@redhat.com 的 T1。让我们听到你的声音！

*Last updated: April 21, 2021*