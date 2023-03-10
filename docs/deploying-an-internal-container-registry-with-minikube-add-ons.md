# 使用 Minikube 附加组件部署内部容器注册中心

> 原文：<https://developers.redhat.com/blog/2019/07/11/deploying-an-internal-container-registry-with-minikube-add-ons>

Minikube 有一个名为 add-ons 的特性，可以帮助向 Minikube 的 Kubernetes 集群添加额外的组件和特性。

*registry* 插件将部署一个内部注册表，然后可以使用它来推和拉 Linux 容器映像。但是有时，我们可能希望模仿对不同注册中心的推送和拉取(例如，对容器注册中心使用别名)。在本文中，我将带您完成实现相同目标所需的步骤。

## 我们需要什么？

*   Minikube，首选版本是 v1.1.1
*   Git 克隆源代码
*   [库贝克特尔](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
*   [库本斯](https://github.com/ahmetb/kubectx/blob/master/kubens)
*   [tkn CLI](https://github.com/tektoncd/cli)

## 我们要做什么？

作为本练习的一部分，我们将:

*   通过 Minikube 附件启用注册表。
*   更新域 dev.local，example.com 的 Minikube 节点的 etc/hosts，以解析到内部注册表。
*   将 CoreDNS 更新为允许 pods 使用别名将图像(CI/CD 的典型情况)推送到注册表的规则。

## 部署容器注册表

如前所述，我们可以使用 Minikube 附加组件来部署和启用内部注册表。默认情况下，内部注册表部署在`kube-system`名称空间中。

```
minikube profile demo
minikube start -p demo --memory=8192 --cpus=6 --disk-size=50g
```

**注意:**如果你喜欢用 [cri-o](https://cri-o.io/) ，那么把上面的命令调整成这样:

```
minikube profile demo
minikube start -p demo --memory=8192 --cpus=6 --container-runtime=crio

```

Minikube 运行后，下一步是部署注册表。

```
minikube addons enable registry
```

一旦注册中心被启用，您将在 kube-system 名称空间中看到一个注册中心 pod `kubectl -n kube-system get pod`和一个相应的服务`kubectl -n kube-system get svc`。

GitHub 中的 [minkube-helper](https://github.com/kameshsampath/minikube-helpers) repo 拥有源代码以及示例应用程序来测试配置。克隆源代码并导航到`registry`子文件夹。为了便于参考，我们将 sources 文件夹命名为$PROJECT_HOME:

```
git clone https://github.com/kameshsampath/minikube-helpers && \
cd minikube-helpers/registry
```

## 创建别名配置映射

我们希望用于注册表的别名是通过 ConfigMap 配置的，称为`registry-aliases`:

```
apiVersion: v1
data:
  # Add additonal hosts seperated by new-line
  registryAliases: >-
    dev.local
    example.com
  # default registry address in minikube when enabled via minikube addons enable registry
  registrySvc: registry.kube-system.svc.cluster.local
kind: ConfigMap
metadata:
  name: registry-aliases
  namespace: kube-system

```

```
kubectl apply -f registry-aliases-config.yaml
```

## 更新 Minikube /etc/hosts 文件

为了将别名解析到`kube-system`名称空间中的`registry`服务，我们需要在 Minkube 虚拟机的`/etc/hosts`文件中添加别名条目。我们将使用 [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 来更新 Minikube 虚拟机中的`etc/hosts`文件。

```
kubectl apply -f node-etc-hosts-update.yaml
```

由于运行 DaemonSet 需要几分钟时间，因此您可以使用以下命令来查看状态:

```
kubectl -n kube-system get pods --watch
```

一旦 DaemonSet 成功运行，您就可以检查 Minkube VM 的`/etc/hosts`文件，该文件现在将被更新为指向注册表服务的 CLUSTER-IP。

```
minikube ssh -- cat /etc/hosts
```

### 技巧

*   您可以使用 CTRL+C 来终止手表。
*   您可以使用命令`kubectl -n kube-system get svc registry -o jsonpath='{.spec.clusterIP}'`检查`registry`服务的 CLUSTER-IP。

## 补丁核心

我们在上一节中应用的配置和其他设置对于容器运行时来说足够好，可以推和拉图像。典型的 CI/CD 场景类似于 Kubernetes pod 进行构建，例如 Jenkins、Tekton，并将容器映像作为管道的一部分推送到注册站。

为了让 pod 解析别名，如 dev.local、example.com，我们需要配置 [CoreDNS](https://coredns.io/) 规则。为了让我们的别名配置工作，我们将使用 CoreDNS [重写](https://coredns.io/plugins/rewrite/)规则。

运行以下命令将使用重写规则修补核心 DNS。

```
./patch-coredns.sh
```

可以通过查询 CoreDNS 补丁来查看更新。成功的更新将显示命令`kubectl -n kube-sytem configmap coredns -oyaml`的输出，如下所示:

```
apiVersion: v1
data:
  Corefile: |-
    .:53 {
        errors
        health
    rewrite name dev.local  registry.kube-system.svc.cluster.local
    rewrite name example.com  registry.kube-system.svc.cluster.local
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
```

## 测试配置

我发现这个注册表黑客的真正需求是在我试图部署 Tekton 管道的时候。Tekton 是 Kubernetes 声明 CI/CD 管道的本地方式。

作为我的管道的一部分(参见[示例](https://github.com/kameshsampath/minikube-helpers/tree/master/registry/example)，我想构建和部署一个简单的 Hello World 应用程序。

## 部署 Tekton 管道

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/latest/release.yaml
```

Tekton 管道的状态可通过以下方式观察:

```
kubectl get pods --namespace tekton-pipelines -w
```

## 部署应用程序管道

```
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/master/golang/build.yaml \
  --filename example/build.yaml
```

由于完成管道需要一些时间，您可以使用以下命令来观察状态:

```
tkn taskrun logs -f -a hello-world
```

成功的管道构建将部署 Hello World 应用程序。您可以使用以下 Minikube 快捷方式:

```
curl $(minikube service helloworld --url)
```

来调用服务，该服务返回“Hello World”作为响应。

**注意:**当你看到`tkn logs -f -a hello-world`显示一个空白屏幕时，可能是它正在拉所需的图像。要知道发生了什么，可以使用`kubectl get events -w`。

我希望这篇文章能对您可能遇到的类似开发环境用例有所帮助。下一次，我们将深入探讨泰克顿。在那之前，快乐的库伯内特黑客！

*Last updated: January 5, 2022*