# kubectl:Kubernetes 命令行的开发技巧

> 原文：<https://developers.redhat.com/blog/2020/11/20/kubectl-developer-tips-for-the-kubernetes-command-line>

Kubectl，即 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 命令行界面(CLI)，拥有比许多开发人员意识到的更多的功能。例如，您知道`kubectl`在集群内部运行时可以访问 Kubernetes API 吗？您还可以使用`kubectl`来假设不同的用户身份，选择一个自定义编辑器来运行`kubectl edit`命令，等等。

在本文中，我将介绍几个能够改善您日常工作流程的 CLI 特性。您还将在 [Kubernetes 1.20](https://www.kubernetes.dev/resources/release/) 中了解新的`kubectl debug`命令。

## 集群内配置

当`kubectl`需要定位一个配置文件时，它会检查几个地方。您可能熟悉`$HOME/.kube/`目录，这是默认目录，`kubectl`存储其配置和缓存文件。您还听说过`--kubeconfig`标志，或`KUBECONFIG`环境变量，它用于传递配置文件的位置。

`kubectl`在加载文件时检查的另一个位置是集群内配置。没有多少用户知道这个选项，所以我将用一个例子来演示它是如何工作的。

### 从容器中取出豆荚

首先，我们将运行一个简单的`centos:7`容器图像:

```
$ kubectl run centos --stdin --tty --image=centos:7

```

我传递了`--stdin`和`--tty`标志，以便在 pod 一运行就将其附加到 pod 上。我们还需要一个`kubectl`二进制吊舱:

```
$ kubectl cp kubectl centos:/bin/

```

现在，让我们看看在 CentoOS 7 容器上尝试一个`get pods`命令会发生什么:

```
$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:soltysh:default" cannot list resource "pods" in API group "" in the namespace "soltysh"

```

此错误表明我的用户没有执行此操作的必要权限。如果您仔细查看用户名(`system:serviceaccount:soltysh:default`)，您会注意到它实际上是分配给我的 pod 的默认服务帐户。但是这是从哪里来的呢？为了找到答案，我们可以增加`kubectl`的详细程度并检查调试信息:

```
$ kubectl get pods -v=4
I1104 11:51:21.969428      57 merged_client_builder.go:163] Using in-cluster namespace

```

注意，我在这个命令中使用了[集群内配置](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#accessing-the-api-from-a-pod)。集群内配置检查位于`/var/run/secrets/kubernetes.io/serviceaccount/token`中的服务帐户令牌。它还检查两个环境变量`KUBERNETES_SERVICE_HOST`和`KUBERNETES_SERVICE_PORT`。当它找到所有这三个时，它知道它正在 Kubernetes 集群内部运行。然后，它知道应该读取注入的数据来与集群通信。

### 使用查看角色进行读取访问

大多数默认的 Kubernetes 集群打开了[基于角色的访问控制](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) (RBAC)，拥有一组[预先存在的集群角色](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles)。这里，我们将使用`view`角色，它拥有对所有非升级资源的读取权限。要使用这个角色，我们需要创建一个`ClusterRoleBinding`，就像这样:

```
$ kubectl create clusterrolebinding view-soltysh --clusterrole=view --serviceaccount=soltysh:default

```

现在，当我们重新运行`get pods`命令时，我们应该能够查看在当前名称空间中运行的所有 pod:

```
$ kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
centos   1/1     Running   0          18m

```

操作成功是因为我们使用了一个允许我们查看窗格的角色。

**注意**:如果您想知道名称空间缺省发生在哪里，请检查`/var/run/secrets/kubernetes.io/serviceaccount/namespace`文件的内容。

## 带有`--as=user`标志的用户模拟

Kubernetes 具有类似于 Unix 的`sudo`命令的功能。这个特性叫做[用户模拟](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)，可以让你以不同用户的身份调用任何命令。要在`kubectl`中使用这个特性，您需要指定`--as=user`标志，其中`user`是您希望模拟的用户的名字。同样，一个例子将证明这个概念。

### 允许`impersonate`

要设置演示，让我们从一个非集群管理员用户开始。我们将尝试从上一个示例中使用的名称空间中获取窗格:

```
$ KUBECONFIG=nonadmin kubectl get pods -n soltysh
Error from server (Forbidden): pods is forbidden: User "nonadmin" cannot list resource "pods" in API group "" in the namespace "soltysh"

```

不出所料，我们收到一个错误消息，指出我们无权访问该名称空间。现在，让我们使用`--as=system:admin`标志尝试相同的命令。这让我们可以模拟`system:admin`:

```
$ KUBECONFIG=nonadmin kubectl get po -n soltysh --as=system:admin
Error from server (Forbidden): users "system:admin" is forbidden: User "nonadmin" cannot impersonate resource "users" in API group "" at the cluster scope

```

这个错误告诉我们，我们没有用户模拟所需的权限。也就是说，我们需要[在用户属性](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)上执行“impersonate”动作，但是我们目前没有权限去做这件事。我们如何访问我们需要的权限？

### `impersonator`集群角色

在我的集群中，我有一个`impersonator`集群角色。要使用`impersonate`动词，我们需要将这个角色分配给当前用户。诀窍是调用`impersonator`操作作为`cluster-admin`:

```
$ kubectl create clusterrolebinding impersonator-nonadmin --clusterrole=impersonator --user=nonadmin

```

现在，当我们作为非管理员用户重新运行该命令时，我们可以从`soltysh`的名称空间中读取 pod:

```
$ KUBECONFIG=nonadmin kubectl get po -n soltysh --as=system:admin
NAME     READY   STATUS    RESTARTS   AGE
centos   1/1     Running   0          36m

```

为了证实这个技巧有效，让我们看看当我们尝试获取没有`--as`标志的 pod 时会发生什么:

```
$ KUBECONFIG=nonadmin kubectl get po -n soltysh
Error from server (Forbidden): pods is forbidden: User "nonadmin" cannot list resource "pods" in API group "" in the namespace "soltysh"

```

这个命令再次失败。只有当我们正确使用`--as`标志时，模拟才有效。

**注意**:`kubectl`CLI 还可以访问`--as-group`标志，这允许您模拟一个组。此外，您可以通过将`-v=8`与您的`kubectl`命令一起使用来验证该命令是如何工作的。这样做可以让您查看发送到集群的所有标头。

## 指定自定义编辑器

默认情况下，`kubectl edit`命令假设您正在使用`vi`(在 [Unix 风格的系统](https://developers.redhat.com/topics/linux)上)或记事本(在 [Windows](https://developers.redhat.com/blog/category/windows) 中)作为您的编辑器。如果您喜欢不同的编辑器，您可以使用`KUBE_EDITOR`环境变量来指定它:

```
$ KUBE_EDITOR="code --wait" kubectl edit po/centos -n soltysh --as=system:admin

```

在这种情况下，我使用的是 [Visual Studio 代码](https://developers.redhat.com/blog/category/vs-code) (VS 代码)。注意，我已经指定了`--wait`标志，以确保 VS 代码在所有编辑操作完成之前保持控制权。

## 调试正在运行的应用程序

我们要看的最后一个命令是来自 Kubernetes 1.19 的`alpha`。您可以使用此命令来调试正在运行的应用程序。

`kubectl alpha debug`命令是为[短暂容器](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/)开发的，但后来已经转变成一个成熟的调试工具。您可以使用`kubectl alpha debug`创建以下任何内容:

*   用于调试应用程序的临时容器(假设启用了该功能)。
*   带有附加工具的 running pods 副本，以便在应用程序出现故障时提供洞察力。
*   可以用来调试节点的 pod。

`kubectl alpha debug`命令有更多的特性供您检查。此外， [Kubernetes 1.20 将该命令升级为 beta](https://www.kubernetes.dev/resources/release/) 。如果你在 Kubernetes 1.20 中使用`kubectl` CLI，你会在`kubectl debug`下找到`alpha`命令。

## 结论

希望我在这篇文章中分享的小技巧对你的日常工作有所帮助。我将给你们留下四点启示:

1.  `kubectl`命令知道如何使用集群内配置来与运行它的集群通信。您需要确保您对分配给您的 pod 的服务帐户拥有适当的[访问权限](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)。
2.  对于基于 Unix 的系统，`kubectl` `--as`标志的作用类似于`sudo`。您需要对[模拟动词](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#user-impersonation)拥有适当的访问权限。
3.  `KUBE_EDITOR`允许您为`kubectl edit`命令选择不同的编辑器。
4.  在 Kubernetes 1.20 和更高版本中，尝试使用新的`kubectl debug`命令来调试应用程序。

如果您对这些工具有任何疑问或改进建议，请联系我或其他 [SIG-CLI 团队成员](https://github.com/kubernetes/community/tree/master/sig-cli)。我们是 Kubernetes 针对 CLI 的特别兴趣小组。

*Last updated: November 19, 2020*