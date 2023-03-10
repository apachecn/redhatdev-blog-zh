# 将 Coolstore 微服务引入服务网格:第 1 部分——探索自动注入

> 原文：<https://developers.redhat.com/blog/2018/04/05/coolstore-microservices-service-mesh-part-1-exploring-auto-injection>

随着行业走向对[云原生微服务](https://developers.redhat.com/topics/microservices/)幻灭的[低谷，](https://www.gartner.com/technology/research/methodologies/hype-cycle.jsp)终于明白分布式架构引入了更多的复杂性(很奇怪，对吧？)， [*服务网格*](https://developers.redhat.com/topics/service-mesh/) 可以帮助软化着陆，将一些复杂性从我们的应用中转移出来，并将其放在它应该在的地方，即应用运营层。

在 Red Hat，我们致力于(并积极参与)上游 [Istio 项目](https://developers.redhat.com/topics/service-mesh/)，并努力将其整合到 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview/) 中，以将服务网络的好处带给我们的客户和更广泛的相关社区。如果你想玩 Istio，查看 learn.Openshift.com 上的[服务网格教程。如果您想安装它，请遵循](https://learn.openshift.com/servicemesh/) [Istio Kubernetes 快速入门说明](https://istio.io/docs/setup/kubernetes/quick-start.html)并将其安装在 Red Hat OpenShift 3.7 或更高版本上(如果您想使用自动注入，则安装在 3.9 版本上)。

## 现有应用即服务网格

[![Istio basic flow](img/d7a7a4187c3007e61407ad2cf6f88f1d.png "Istio basic flow")](/sites/default/files/blog/2018/03/Istio-basic-flow.png)Existing services with sidecars attached">

你可能已经在去年看到了新的 [Coolstore 微服务演示](https://github.com/jbossdemocentral/coolstore-microservice)在红帽生态系统中浮动；这是展示 Red Hat 为现代应用程序带来的独特价值的绝佳工具，并展示了使用 Red Hat 堆栈进行现代应用程序开发和集成的关键用例。如果我们可以使用 Istio 和 Red Hat OpenShift 将现有的应用程序(如 Coolstore)部署为[服务网格，这不是很好吗？毕竟，Istio](https://developers.redhat.com/topics/kubernetes/) 的[目标之一是，它透明地为现有的应用程序带来新的价值，而他们甚至不知道这一点。它可以减少或消除应用程序中处理重试、断路器、TLS 等的大量样板代码。](https://istio.io/docs/concepts/what-is-istio/goals.html)

让我们开始工作，创造酷店。我们假设您已经安装了 Red Hat OpenShift 3.9。我用的是红帽[open shift Origin 3 . 9 . 0 . alpha 3；](https://github.com/openshift/origin/releases/tag/v3.9.0-alpha.3)截至发稿时，Red Hat OpenShift 容器平台 3.9 尚未发布。我们进一步假设您已经安装了 Istio 0.6.0 或更高版本，方法是遵循 Kubernetes 的 [Istio 快速入门。复制 Coolstore repo，然后继续玩下去:](https://istio.io/docs/setup/kubernetes/quick-start.html)

```
% git clone https://github.com/jbossdemocentral/coolstore-microservice
```

并且确保您以集群管理员的身份登录，或者拥有*集群管理*权限，因为这将要求您稍后进行一些策略和权限更改。

## 自动注射边车

有了 sidecar 自动注入，你的应用程序的豆荚会自动用特使代理装饰起来，甚至不需要改变应用程序的部署。这依赖于 Kubernetes 1.9 中新增的 [Kubernetes 的](https://kubernetes.io/docs/admin/admission-controllers/#mutatingadmissionwebhook-beta-in-19)(因此也是 Red Hat OpenShift 3.9)。要在 Red Hat OpenShift 中启用这个功能，您需要编辑您的主配置文件(`master-config.yaml`)来添加`MutatingAdmissionWebhook`:

```
admissionConfig:
  pluginConfig:
    MutatingAdmissionWebhook:
      configuration:
        apiVersion: v1
        disable: false
        kind: DefaultAdmissionConfig
```

并启用 Kubernetes 证书签名 API，以便使用 Kubernetes 对 webhook 证书进行签名(自动边车注入安装过程的一部分):

```
kubernetesMasterConfig:
  controllerArguments:
    cluster-signing-cert-file: [ ca.crt ]
    cluster-signing-key-file: [ ca.key ]

```

有了这些改变，重新启动你的主人，然后按照[自动边车注射安装过程](https://istio.io/docs/setup/kubernetes/sidecar-injection.html#automatic-sidecar-injection)。

请注意，与现成的 Kubernetes 相比，Red Hat OpenShift 的默认安全策略更加严格，因此您必须允许 injector webhook 以提升的权限运行，因为它将尝试绑定到其 pod 中的端口 443。Istio 项目意识到了关于 it 需要太多特权的抱怨，并且正在努力应用最小特权原则:

```
% oc adm policy add-scc-to-user anyuid -z istio-sidecar-injector-service-account -n istio-system
% oc adm policy add-scc-to-user privileged -z istio-sidecar-injector-service-account -n istio-system

```

然后重启注射器网状挂钩盒。当创建新的 pod 来运行应用程序容器时，将会咨询 MutatingAdmissionWebhook，并给它一个机会来更改 pod 的内容。它将添加必要的“sidecar”容器来透明地拦截所有网络流量和所有入站/出站应用程序流量。

接下来，让我们创建一个测试项目来容纳一个示例应用程序。授予代理容器权限，允许它们变魔术，并与特权用户一起运行，以便我们稍后可以 rsh 进入:

```
% oc new-project coolstore-test
% oc adm policy add-scc-to-user privileged -z default,deployer
% oc adm policy add-scc-to-user anyuid -z default,deployer 

```

要在 Red Hat OpenShift 项目上启用自动注入，您只需标记项目(又名名称空间):

```
% oc label namespace $(oc project -q) istio-injection=enabled
```

从那时起，*在那个项目中创建的任何*pod 都将获得一个额外的容器注入其中。让我们通过创建一个运行基本 Apache HTTPD 服务器的测试舱来快速测试一下:

```
% oc new-app httpd
```

看看这些豆荚:

```
% oc get pods
NAME           READY STATUS RESTARTS AGE
httpd-1-deploy 1/2   Error  0        6s
```

看到`READY`栏下面的那个`1/2`了吗？上面说两个集装箱中有一个准备好了。这两个容器是:执行部署的容器；还有那个自动注射的边车。在一个豆荚里有多个容器总是可能的，但迄今为止，它还没有在野外广泛出现。各种各样的开发工具都包含了一些假设，这些工具需要修改才能在一个独立的世界中顺利运行。

请注意，`httpd-1-deploy`窗格没有运行应用程序；运行 Red Hat OpenShift 部署的 pod 正在尝试部署运行应用程序的 pod(通常称为“部署者 pod”)。如您所见，部署的状态是错误。部署者 pod 日志文件揭示了:

```
% oc logs -c deployment httpd-1-deploy
error: couldn't get deployment httpd-1: Get https://172.30.0.1:443/api/v1/namespaces/coolstore-test/replicationcontrollers/httpd-1: dial tcp 172.30.0.1:443: getsockopt: connection refused
```

这是由于 Istio/Envoy 中的一个 [bug。作为一种解决方法，让我们破解它，添加一些睡眠时间，以允许 sidecar 代理在 _actual_ deployment 发生之前有额外的时间进行初始化:](https://github.com/istio/istio/issues/3533)

```
% oc patch dc/httpd -p '{ "spec": { "strategy": { "customParams": { "command": ["/bin/sh", "-c", "sleep 5; echo slept for 5; /usr/bin/openshift-deploy"]}}}}'
deploymentconfig "httpd" patched
```

通常，当您修补部署时，会立即触发新的部署。这给我们带来了下一个问题:以前的部署从来没有“完成”。问题是附加到部署者 pod 的 sidecar 代理还没有退出(为什么会退出？).因此 pod 继续运行，直到这个 pod 完成并且它的容器退出，部署才被认为完成；永远不会(直到 6 小时后超时，此时整个部署将被回滚)。啊！

因此，让我们取消当前正在运行(但失败了)的部署:

```
% oc rollout cancel dc/httpd
deploymentconfig "httpd" cancelling
```

等待 pod 终止，然后再次触发部署:

```
% oc rollout latest httpd
deploymentconfig "httpd" rolled out
```

现在该应用程序应该推出了，因为 sleep 5 将为部署者 pod 提供一些时间来等待 Istio 网络魔法发挥作用。尽管和以前一样，部署者 pod 永远不会退出。过一会儿你会看到:

```
% oc get pods
NAME           READY STATUS    RESTARTS AGE
httpd-2-deploy 1/2   Completed 0        56s
httpd-2-rbwdq  2/2   Running   0        47s
```

一段时间后，您可以看到实际的 HTTPD 应用程序在`httpd-2-rbwdq`窗格中的一个容器内运行，而部署者窗格(`httpd-2-deploy`)仍在运行，因为与部署者窗格相关联的代理从未退出。让我们终止与 deployer pod 相关联的代理，以便完成部署。我们将通过 rsh'ing 进入 deployer pod(指定运行 istio-proxy 的代理容器)来实现这一点，并使用`pkill`终止 istio 代理进程:

```
~ % oc rsh -c istio-proxy httpd-2-deploy pkill -f istio
command terminated with exit code 137
```

然后，您可以运行`oc get pods`和`oc get dc/httpd`来观察应用程序是否正确运行了它的 sidecar 容器:

```
~ % oc get pods
NAME          READY STATUS  RESTARTS AGE
httpd-2-m29d9 2/2   Running 0        1m
~ % oc get dc
NAME  REVISION DESIRED CURRENT TRIGGERED BY
httpd 2        1       1       config,image(httpd:2.4)
```

## 总结和意见

Istio 代理的自动注入是一个引人注目的新特性，它将为您的 Red Hat OpenShift 项目注入新的活力。然而，Red Hat OpenShift 中需要一些微调，以在整个应用程序生命周期中充分利用 Red Hat OpenShift 中与构建和部署应用程序相关的功能。其他观察结果:

*   作为代理初始化的一部分出现的网络魔法似乎暂时切断了与 Red Hat OpenShift 网络的联系；我们用名副其实的睡眠黑客解决了这个问题，但是需要一个更好的解决方案。
*   需要一种更精细的机制来指定哪些容器被自动注射。目前，[是在项目(Kubernetes 名称空间)级别](https://istio.io/docs/setup/kubernetes/sidecar-injection.html#understanding-what-happened)用标签完成的，这意味着在名称空间中创建的 _every_ pod 将被注入一个代理。您还可以使用部署上的`sidecar.istio.io/inject: "true"`注释有选择地禁用每个应用程序的注入。然而，还不清楚这将如何影响代表在 Red Hat OpenShift 中构建或部署的应用程序而创建的特殊*构建器*和*部署器*pod。这个问题的解决方案应该在 Red Hat OpenShift 3.10 中实现。
*   使用自动注入时，一些应用程序的部署可能会失败，并出现一个奇怪的错误`reflect.Value.Addr of unaddressable value`。这是一个 [Go 语言级别的错误](https://github.com/kubernetes/kubernetes/pull/53896)，已经在 Kubernetes 中得到解决，并将出现在 Red Hat OpenShift 的下一个版本中。目前，除了使用手动注入之外，没有其他解决方法，我们将在本系列文章的下一部分中介绍这一点。
*   自动注入对于演示和快速启动现有应用程序并在网格中运行非常有用。然而，在生产场景中，我不确定我是否愿意信任自动注入机制。手动注入允许您做同样的事情，但是将结果提交到您的源代码管理系统，并且不依赖于自动注入。我可能会采用的另一种方法是在一个单独的集群和名称空间中进行构建，根本没有任何自动注入。将注入留给在我的生产集群/命名空间中发生的部署。

所以现在让我们关闭自动注入:

```
% oc label namespace $(oc project -q) istio-injection-
```

末尾的连字符(`-`)表示“删除标签”。

在本系列的下一部分中，我们将向您展示如何进行手动注入(从 Istio 0.6.0 开始，[支持 OpenShift DeploymentConfig](https://github.com/istio/istio/commit/0198fc5ca214fcd7b3e9f15ae471146124bef59c) 对象)，我们将把它应用到整个 Coolstore 项目中以获得一些真正的乐趣。
敬请期待！

* * *

****更新:阅读[第 2 部分-手动注射](https://developers.redhat.com/blog/2018/04/12/bringing-coolstore-microservices-to-the-service-mesh-part-2-manual-injection/)现在****

* * *

*Last updated: September 3, 2019*