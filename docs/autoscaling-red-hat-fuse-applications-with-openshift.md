# 利用 OpenShift 自动缩放 Red Hat 保险丝应用

> 原文：<https://developers.redhat.com/blog/2019/11/11/autoscaling-red-hat-fuse-applications-with-openshift>

在本文中，我们用 [Red Hat Fuse](https://developers.redhat.com/products/fuse/overview) 应用程序演示了 [Red Hat OpenShift 的](http://developers.redhat.com/openshift/)水平自动缩放特性。结果是一个基于 Spring Boot 的应用程序，它使用 Apache Camel 组件 [`twitter-search`](https://camel.apache.org/components/2.x/twitter-search-component.html) 根据特定的关键字在 Twitter 上搜索推文。如果流量或 tweets 的数量增加，并且该应用程序不能满足所有请求，那么该应用程序会通过增加 pods 的数量来自动缩放。通过跟踪特定 pod 上该应用程序的 CPU 利用率来监控为所有请求提供服务的能力。此外，一旦流量或 CPU 利用率恢复正常，pod 的数量就会减少到最小配置值。

缩放有两种类型:*水平*和*垂直*。水平扩展是指增加应用程序实例或容器的数量。垂直扩展是指在运行应用程序或容器的运行时，CPU 和内存等系统资源增加。水平扩展可用于无状态的应用程序的*，而垂直扩展更适合于有状态的*应用程序。**

当应用程序部署在来自任何云供应商的云环境中时，自动伸缩是有用的，用户必须为 CPU 和内存等资源付费。在周末或特定的工作日，甚至在某一天的几个小时内，流量可能会更大，因此静态设置可能会花费更多的成本，并且不能充分利用系统资源。拥有一个应用程序可以自我扩展的动态环境可以降低总体成本。

**注意:**出于演示和概念验证的目的，我们使用的是 Red Hat OpenShift 3.11 和 Red Hat Fuse 7.4。我们的目标是显示 OpenShift 特性之一(自动缩放)，这是传统应用程序所不具备的。

### 先决条件

按照官方[红帽保险丝文档](https://access.redhat.com/documentation/en-us/red_hat_fuse/7.4/html/fuse_on_openshift_guide/get-started-admin#configure-container-registry)中的说明在 OpenShift 上设置红帽保险丝 7.4。OpenShift 应该根据从 pod 收集的指标，自动增加或减少复制控制器或部署配置的规模。为了从您的 pods 收集这些 CPU 和内存指标，必须在您的 OpenShift 服务器上安装`metrics`服务器，以从 pods 获取 CPU 和内存指标。查看 [Pod 自动缩放指南](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html),了解如何使用 Ansible 实现这一点。

## 创建应用程序

这个演示的代码可以在[我的个人 GitHub 库](https://github.com/1984shekhar/camel-twitter-shift)中找到。首先使用源到映像(S2I)方法将这段代码部署到您的 OpenShift 环境中。首先，从 OpenShift 目录中选择一个应用程序模板。我选择了红帽保险丝 7.4 骆驼与 Spring Boot，如图 1 所示:

[![The OpenShift catalog with an application template selected.](img/d9b8aac69a3700b72679140c5e9eb8a1.png "1_Catalog_Fuse")](/sites/default/files/blog/2019/10/6_Catalog_Fuse.png)*Figure 1: The OpenShift catalog with an application template selected.*">

接下来，将 Git 存储库 URL 指向 GitHub 存储库的链接。在我们的例子中，这个`camel-twitter-shift`组件使用关键词`imcdemo,` `RED_HAT_APAC`或`RedHatTelco`搜索 tweets。该组件将在两分钟内轮询 Twitter，这是用`delay`属性和其他特定于用户的属性配置的。

现在，将 Git 引用指向包含实际代码的分支。我将其设置为`master`，如图 2 所示:

[![Configuring the selected application template.](img/9941d2210770f81b55a46428e3bf3a69.png "7_s2i_github_hook")](/sites/default/files/blog/2019/10/7_s2i_github_hook.png)*Figure 2: Configuring the selected application template.*">

通过 [Twitter 开发者平台](https://developer.twitter.com/en/docs/basics/getting-started)，让你的`consumerKey`、`consumerSecret`、`accessToken`和`accessTokenSecret,`注册标准 API(免费，但有限制)。在使用 tweets 搜索关键字后，我们使用 Camel log 组件将它们记录到控制台:

```
<route id="twitter-search">
<from id="route-search" uri="twitter-search://imcdemo OR RED_HAT_APAC OR RedHatTelco?type=polling&amp;delay=120000&amp;consumerKey=dF&amp;consumerSecret=Ay&amp;accessToken=7d&amp;accessTokenSecret=9K"/>
<log id="route-log-search" message=">>> ${body}"/>
</route>

```

## 设置水平自动缩放

我们可以使用下面的命令为`fuse74-camel-twitter`部署配置设置自动缩放。请注意，`cpu-percent`设置为 10，这意味着一旦 pod 的 CPU 利用率超过 10%，应用程序将扩展新的 pod，pod 的最大数量为 5:

```
cpandey@cpandey camel-twitter-shift]$ oc autoscale dc/fuse74-camel-twitter --min 1 --max 5 --cpu-percent=10 horizontalpodautoscaler.autoscaling/fuse74-camel-twitter autoscaled 

```

在图 3 中，第一个红色箭头指向配置的 pod 的最小和最大数量。第二个箭头是我们可以从外部基于 HTTP 的服务或应用程序访问 pod 的路线。这个设置只是将 HTTP 流量发送到 pod:

[![Setting up autoscaling.](img/a94d3328fd4a97ee1bb4dc6f6edab030.png "1_Standard_GUI_view")](/sites/default/files/blog/2019/10/1_Standard_GUI_view.png)POD-Image3*Figure 3: Setting up autoscaling.*">

接下来，让我们从水平 pod 自动缩放器(HPA)获取信息:

```
[cpandey@cpandey camel-twitter-shift]$ oc get hpa

NAME REFERENCE TARGETS MINPODS MAXPODS REPLICAS AGE

fuse74-camel-twitter DeploymentConfig/fuse74-camel-twitter 10% 1 5 0 4s
```

我们可以在 OpenShift 的 GUI 中查看 pod 的数量，如图 4 所示:

[![The number of pods as shown in the OpenShift GUI.](img/0979e4d93456ab214587732daeb0b648.png "POD_CLI-Image4")](/sites/default/files/blog/2019/10/2_Standard_CLI_view.png)*Figure 4: The number of pods as shown in the OpenShift GUI.*">

但是，我发现命令行提供的信息更多，因为在运行时它提供了 CPU 利用率和活动 pod 的数量:

```
[cpandey@cpandey camel-twitter-shift]$ watch 'oc describe hpa fuse74-camel-twitter'
```

您可以在 [OpenShift Pod 自动缩放文档](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html)中找到关于此命令的更多信息。

## 模拟应用程序的流量

为了模拟流量，以便我们可以演示 OpenShift autoscaler 特性，有另一个接受 HTTP 请求的路由。这条路线使用 Camel 的回流`component`，它启动了`undertow` web 容器。该容器监听端口 9956:

```
<route id="load-route">
    <from id="_from1" uri="undertow:http://0.0.0.0:9956/undertowTest"/>
    <convertBodyTo id="_convertBodyTo1" type="java.lang.String"/>
    <log id="_log1" loggingLevel="INFO" message="\n REQUEST RECEIVED :\n Headers: ${headers}\n Body: ${body} \n"/>
    <setBody id="_setBody1">
    <constant>hello all</constant>
    </setBody>
</route>
```

使用`loop.sh`脚本，我们可以向这个路由批量发送请求。我们可以在三个或四个终端上运行这个脚本，这样就会有并发流量，这将会快速提高 CPU 利用率。这个脚本也是项目的一部分:

```
#!/bin/bash
for ((;;))
do
curl http://demo-service-imc-demo.apps.csppnq.lab.pnq2.cee.redhat.com/undertowTest
done
```

GUI 中的结果见图 5，命令行版本见图 6:

[![Loop script results in the GUI.](img/f20a85921f59c818a38628bba5f36503.png "4_AutoScaline_GUI_view")](/sites/default/files/blog/2019/10/4_AutoScaline_GUI_view.png)AutoScaling_to_3_pods-Image5*Figure 5: Loop script results in the GUI.*">[![Loop script results on the command line.](img/0f4f578c906fb862dff90b82fc763762.png "3_AutoScaling_CLI_view_increment")](/sites/default/files/blog/2019/10/3_AutoScaling_CLI_view_increment.png)AutoScaling_to_3_pods-Image6*Figure 6: Loop script results on the command line.*">

如果我们停止执行这些脚本，那么 CPU 利用率将恢复正常:

[![After stopping the loop script.](img/8bddba9ff2d0fae4d7f439007529f5fd.png "5_AutoScaling_CLI_view_decrement")](/sites/default/files/blog/2019/10/5_AutoScaling_CLI_view_decrement.png)AutoScaling_to_1_pods-Image7*Figure 7: After stopping the loop script.*">

## 禁用水平窗格自动缩放

我们可以使用以下命令禁用自动缩放:

```
[cpandey@cpandey camel-twitter-shift]$ oc delete hpa fuse74-camel-twitter

horizontalpodautoscaler.autoscaling "fuse74-camel-twitter" deleted
```

就是这样。我希望这个演示可以帮助你设置和使用 OpenShift 的 autoscaler 特性，以及 Red Hat Fuse 应用程序或任何其他应用程序。

*Last updated: July 1, 2020*