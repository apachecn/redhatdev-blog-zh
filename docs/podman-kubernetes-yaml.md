# 波德曼现在可以轻松过渡到 Kubernetes 和 CRI-O

> 原文：<https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml>

配置 [Kubernetes](https://developers.redhat.com/blog/category/kubernetes/) 是在 YAML 文件中定义对象的一个练习。虽然不是必需的，但拥有一个至少能理解 YAML 语的编辑器是很好的，如果它懂 Kubernetes 语言就更好了。库伯内特 YAML 是描述性的和强有力的。我们喜欢用声明性语言对期望状态进行建模。也就是说，如果你习惯了像`podman run`这样简单的东西，过渡到 YAML 的描述可能是一颗难以下咽的苦药丸。

随着 [Podman](https://github.com/containers/libpod) 的继续开发，我们有了更多关于开发者用例和开发者工作流的讨论。这些对话是由用户对我们各种原则的反馈推动的，很明显，容器运行时和技术的激增让一些用户摸不着头脑。最近的一次对话围绕着编排，特别是本地编排。然后斯科特·麦卡蒂抛出了一个想法:“我真正想做的是帮助用户从波德曼到用 Kubernetes 编排他们的容器。”就这样，众所周知的灯泡亮了。

最近对 libpod 的 pull 请求已经开始实现这个想法。请继续阅读，了解更多信息。

Podman 现在可以捕获本地 pod 和[容器](https://developers.redhat.com/blog/category/containers/)的描述，然后帮助用户过渡到更复杂的编排环境，如 Kubernetes。我认为 Podman 是第一个认真对待这种场景并且不依赖于第三方工具的容器运行时。

在考虑如何实现这样的东西时，我们考虑了以下开发人员和用户工作流:

*   在命令行上使用 Podman 在本地创建容器/pod。
*   在本地或在本地化的容器运行时(在不同的物理机器上)验证这些容器/pod。
*   使用 Podman 对容器和 pod 描述进行快照，并帮助用户在 Kubernetes 中重新创建它们。
*   用户可以在快照描述中增加复杂性和协调性(Podman 做不到),并利用 Kubernetes 的高级功能。

## 逗够了；给我看看货物

此时，看一个快速演示可能更有意义。首先，我将创建一个运行`nginx`的简单的 Podman 容器，并将主机的端口 8000 绑定到容器的端口 80。

```
$ sudo podman run -dt -p 8000:80 --name demo quay.io/libpod/alpine_nginx:latest
51e14356dc3f2baad3acc0706cbdb9bffa3cba8c2064ef7db9f8061c77db2ae6
$ sudo podman ps
CONTAINER ID  IMAGE                  COMMAND           CREATED    STATUS        PORTS             NAMES
51e14356dc3f  quay.io/libpod/alpine_nginx:latest  nginx -g daemon o... 4 seconds ago Up 4 seconds ago  0.0.0.0:8000->80/tcp demo
$ curl http://localhost:8000
podman rulez
$

```

如您所见，我们能够成功地`curl`绑定端口，并从`nginx`容器获得响应。现在，我们可以执行将生成 Kubernetes YAML 的容器的快照。因为我们是在网络上测试这个，所以我们也会要求 Podman 生成一个服务文件。

```
$ sudo podman generate kube demo  > demo.yml
```

一旦我们有了 YAML 文件，我们就可以在 Kubernetes 中重新创建容器/容器:

```
$ sudo kubectl create -f demo.yml
pod/demo-libpod created
service/demo-libpod created
```

现在我们可以检查结果，看看是否“波德曼规则”

```
$ sudo kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
demo-libpod   1/1     Running   0     27s
$ sudo kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
demo-libpod   NodePort    10.96.185.205   <none>     80:31393/TCP   24s
kubernetes    ClusterIP   10.96.0.1     <none>        443/TCP        8m17s
$ curl http://192.168.122.123:31393
podman rulez

```

## 波德曼作为集装箱工程师的角色

当在早期开发阶段向人们提出这个想法时，我们真的需要阐明波德曼的角色。有人担心范围蔓延，有人在讨论波德曼的“终点”和 Kubernetes 的起点。最重要的是，我们不想模糊界限，让开发者感到困惑。我们一直依赖于这样一个用例，即开发人员应该使用 Podman 在本地开发容器内容，然后将其“提升”到 Kubernetes 环境中，在那里可以应用对编排的更大控制。

总的来说，这些规则适用于基于当前 Podman 代码和特性的用例:

*   所有生成的对 Podman 容器的描述都“包装”在 Kubernetes V1 Pod 对象中。
*   包含容器的 Podman pod 的所有生成描述也会导致 Kubernetes v1 Pod 对象。
*   NodePort 在服务文件生成中用作向网络公开服务的一种方式。在此过程中会生成一个随机端口。

还应该注意的是，与 Kubernetes 相比，Podman 可以更精细地描述容器。这导致在生成容器的 YAML 时，容器被剥离了一点。此外，因为 Podman 和 Kubernetes 环境实际上并不相连，所以 YAML 的生成不能特定于某些东西，如 IP 地址。前面，我描述了我们使用 NodePort 在 Kubernetes 环境中公开服务。我们显然没有检查随机端口是否真的可用，从精神上来说，这与手工创建库伯内特 YAML 没有什么不同。

## 一个更深入的例子

我最近创建了一个多容器演示，它使用了一个数据库、一个数据库客户机和一个 web 前端。这个想法是为了展示多个容器如何协调工作。它使用三个图像，您也可以使用:

*   `quay.io/baude/demodb:latest` - MariaDB 服务器
*   `quay.io/baude/demogen:latest` -生成随机数并将它们添加到数据库中
*   `quay.io/baude/demoweb:latest` -绘制随机数的 Web 前端

这些容器的工作流程是，`demogen`容器将随机数插入 MariaDB 服务器。然后，用户可以在连接到 web 前端容器时用浏览器查看这些数字的实时图形。

### 与波德曼一起运行演示

在我们最终生成 Kubernetes YAML 之前，我们需要用 Podman 启动并运行演示。在这种情况下，我将在每个“组件”自己的 pod 中运行它们。首先，我们运行 MariaDB，为了简单起见，我将使用`podman container runlabel`。`runlabel`子命令允许 Podman 执行嵌入在容器图像标签中的预定义命令，这通常比键入冗长的命令行选项更容易。当执行`runlabel`命令时，您将看到它实际运行的命令。

```
$ sudo podman container runlabel -p pod quay.io/baude/demodb:latest
Command: /proc/self/exe run --pod new:demodb -P -e MYSQL_ROOT_PASSWORD=x -dt quay.io/baude/demodb:latest
6f451c893e0082960f699967c7188e7dedab98f249600bcb9ccfe0f54368602f
$ sudo podman container runlabel -p pod quay.io/baude/demogen:latest
Command: /proc/self/exe run -dt --pod new:demogen quay.io/baude/demogen:latest python3 generatenums.py
97a8f008686784def0c19bd4314e71bc26209f647d9ea7222940f04fb9eac1c0
$ sudo podman container runlabel -p pod quay.io/baude/demoweb:latest
Command: /proc/self/exe run --pod new:demoweb -dt -p 8050:8050 quay.io/baude/demoweb:latest
5f3096dc54bb6311d75aa604be10e30b6dd804d5bf04816bb85d2368d95274eb

```

我们现在有三个名为`demodb`、`demogen`和`demoweb`的吊舱。每个 pod 包含一个“worker”容器和一个“infra”容器。

```
$ sudo podman pod ps
POD ID         NAME      STATUS    CREATED         # OF CONTAINERS   INFRA ID
5879b590d43e   demoweb Running   4 minutes ago 2              a08da7cc037e
50d18daf6438   demogen Running   4 minutes ago 2              7d8ff6ee1c8a
5aed9c3bbbd5   demodb    Running   4 minutes ago   2     000140e3ebc3
```

请记住，使用 Podman、端口分配、cgroups 等。被分配给“infra”容器，并被 pod 的所有容器继承。记住`podman ps`的默认选项不显示“infra”容器。这需要使用`podman ps`中的`--infra`命令行开关。

```
$ sudo podman ps -p
CONTAINER ID  IMAGE                COMMAND               CREATED        STATUS            PORTS  NAMES            POD
5f3096dc54bb  quay.io/baude/demoweb:latest  python3 /root/cod... 4 minutes ago  Up 4 minutes ago     jovial_joliot      5879b590d43e
97a8f0086867  quay.io/baude/demogen:latest  python3 /root/cod... 4 minutes ago  Up 4 minutes ago     optimistic_joliot  50d18daf6438
6f451c893e00  quay.io/baude/demodb:latest   docker-entrypoint... 4 minutes ago  Up 4 minutes ago     boring_khorana     5aed9c3bbbd5
```

如果您使用 web 浏览器，您可以连接到 web 客户端，查看数据库中随机数的实时绘图。假设您使用了`podman container runlabel`，web 客户端现在被绑定到主机的 8050 端口，并且可以在 http://localhost:8050 上进行解析。

[![Sample output from generated random numbers](img/b5750456507a86b4cea2fc0b22799576.png "graph")](/sites/default/files/blog/2019/01/graph.png)

Sample output from generated random numbers

### 测试 Kubernetes

为了测试 Kubernetes，我一直在 Fedora 29 上使用 [minikube](https://kubernetes.io/docs/setup/minikube/) 、 [libvirt](https://developers.redhat.com/search?t=libvirt) 和 [crio](https://kubernetes.io/docs/setup/minikube/#cri-o) 。出于简洁的目的，我不会深入设置，但以下是我用来启动 minikube 的命令行选项。

```
$ sudo minikube start --network-plugin=cni --container-runtime=cri-o --vm-driver kvm2 --kubernetes-version v1.12.0
```

注意:一些用户不得不添加`--cri-socket=/var/run/crio/crio.sock`来避免 Docker 与 minikube 一起使用。

### 从现有的波德曼豆荚生产库伯内特 YAML

为了从 Podman pod(或 pod 外的容器)生成 Kubernetes YAML 文件，我们使用最近添加的`podman generate kube`命令。请注意创建服务的单数选项。

```
$ sudo podman generate kube --help
NAME:
   podman generate kube - Generate Kubernetes pod YAML for a container or pod

USAGE:
   podman generate kube [command options] CONTAINER|POD-NAME

DESCRIPTION:
   Generate Kubernetes Pod YAML

OPTIONS:
   --service, -s  only generate YAML for kubernetes service object
```

要在 Kubernetes 中运行我们的演示，我们需要为每个 pod 生成 Kubernetes YAML。让我们先为`demodb` pod 生成 YAML。

```
$ sudo podman generate kube demodb
# Generation of Kubernetes YAML is still under development!
#
# Save the output of this file and use kubectl create -f to import
# it into Kubernetes.
#
# Created with podman-0.12.2-dev
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: 2018-12-11T15:24:09Z
  labels:
    app: demodb
  name: demodb
spec:
  containers:
  - command:
    - docker-entrypoint.sh
    - mysqld
    env:
    - name: PATH
      value: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    - name: TERM
      value: xterm
    - name: HOSTNAME
    - name: container
      value: podman
    - name: GOSU_VERSION
      value: "1.10"
    - name: GPG_KEYS
      value: "199369E5404BD5FC7D2FE43BCBCB082A1BB943DB \t177F4010FE56CA3336300305F1656F24C74CD1D8
        \t430BDF5C56E7C94E848EE60C1C4CBDCDCD2EFD2A \t4D1BB29D63D98E422B2113B19334A25F8507EFA5"
    - name: MARIADB_MAJOR
      value: "10.3"
    - name: MARIADB_VERSION
      value: 1:10.3.11+maria~bionic
    - name: MYSQL_ROOT_PASSWORD
      value: x
    image: quay.io/baude/demodb:latest
    name: boringkhorana
    ports:
    - containerPort: 3306
      hostPort: 38967
      protocol: TCP
    resources: {}
    securityContext:
      allowPrivilegeEscalation: true
      privileged: false
      readOnlyRootFilesystem: false
    tty: true
    workingDir: /
status: {}
```

如您所见，我们生成了一个 Kubernetes pod，其中只有一个容器。该容器运行`quay.io/baude/demodb:latest`映像并保留相同的默认命令。现在让我们为每个 pod 创建 YAML，并将结果传输到一个文件中供以后使用。

```
$ sudo podman generate kube demodb > /tmp/kube/demodb.yml
$ sudo podman generate kube demogen > /tmp/kube/demogen.yml
$ sudo podman generate kube demoweb -s > /tmp/kube/demoweb.yml
```

假设我们想要与`demoweb`窗格交互并查看实时图形，我们将为`demoweb`窗格生成一个服务。这将使用 Kubernetes 节点端口将`demoweb` pod 暴露给网络。

### 在 Kubernetes 创建 pod 和服务

我们已经捕获了每个 Podman 吊舱作为 YAML，我们也有一个服务描述。我们可以使用这些文件通过`kubectl`命令创建 Kubernetes pods。

```
$ sudo kubectl create -f /tmp/kube/demodb.yml

pod/demodb created
```

我喜欢在创建其余对象之前运行 MariaDB 容器。

```
$ sudo kubectl get pods
NAME     READY   STATUS    RESTARTS   AGE
demodb   1/1     Running   0     44s
```

现在我们可以创建包括服务在内的其余对象。

```
$ sudo kubectl create -f /tmp/kube/demogen.yml
pod/demogen created
$ sudo kubectl create -f /tmp/kube/demoweb.yml
pod/demoweb created
service/demoweb created
```

我们现在可以看到我们有三个 pod 在运行。

```
$ sudo kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
demodb    1/1     Running   0          2m29s
demogen   1/1     Running   0          61s
demoweb   1/1     Running   0          42s
```

为了连接到实时图，我们需要获取`demoweb` pod 的节点地址和服务的节点端口分配。

```
$ sudo kubectl describe pod demoweb | grep Node:
Node:               minikube/192.168.122.240
$ sudo kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
demoweb      NodePort    10.99.237.80   <none>        8050:32272/TCP   12m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          23m
```

可以通过 http://192.168.122.240:32272 访问`demoweb`客户端。

## 库伯内特 YAML 当地重播

你现在也可以在波德曼“播放”库伯内特 YAML 的文件。我们只支持运行波德曼已经产生的 YAML。这部分是因为我们不能现实地实现整个 Kubernetes 栈。然而，这涵盖了几个有趣的用例。

第一个用例是能够以最少的用户输入重新运行一组本地编排的容器和 pod。在这里，我们考虑容器开发人员能够为迭代开发之类的事情一致地重复以前的容器运行。在相同的上下文中，理论上您也可以在一台机器上工作，并希望在另一台机器上运行相同的容器和 pod。使用 Kubernetes YAML 重新“玩”的能力解决了这两个问题。输入`podman play kube`:

```
$ sudo podman play kube --help
NAME:
   podman play kube - Play a pod based on Kubernetes YAML

USAGE:
   podman play kube [command options] kubernetes YAML file

DESCRIPTION:
   Play a Pod and its containers based on a Kubrernetes YAML

OPTIONS:
   --authfile value             Path of the authentication file. Default is ${XDG_RUNTIME_DIR}/containers/auth.json. Use REGISTRY_AUTH_FILE environment variable to override.
   --cert-dir pathname          pathname of a directory containing TLS certificates and keys
   --creds credentials          credentials (USERNAME:PASSWORD) to use for authenticating to a registry
   --quiet, -q                  Suppress output information when pulling images
   --signature-policy pathname  pathname of signature policy file (not usually used)
   --tls-verify                 require HTTPS and verify certificates when contacting registries (default: true)
```

假设我们在前面的示例应用程序运行之后进行了清理，并且没有定义现有的 pod 或容器。我们可以使用之前生成的 Kubernetes YAML 和`podman play kube`命令，用三个简单的命令重新创建我们之前运行的应用程序。首先，我们验证没有 pod 正在运行:

```
$ sudo podman pod ps
```

现在使用`podman play kube`命令，我们可以基于我们的 Kubernetes YAML 部署吊舱。

```
$ sudo podman play kube /tmp/kube/demodb.yaml
2ff8097fcdc69e4a272a6057ec050a5cb1b872340c31098f23d2e0074a098dbc
4d0e218297156f6d59554ec395ca4651b08d69ab7ca3935ca9f4b2de737cdc14
$ sudo podman play kube /tmp/kube/demogen.yaml
560e4e44e0ae9e43a965ab8da332f3ed89fed1702b4fd14072a5f58a4f133541
bc8c180bfb4616c981f3d1a4c54e1c62ec1a19fb58eacc58040d25a825cbe4e5
$ sudo podman play kube /tmp/kube/demoweb.yaml
ae47ad61e0c5a156dd8dbf4d18700138d5de4693eec63c29835daf765399faac
f01152a2190da8d54f4090eaf75df54faf74df933404ff7fe38026be7f7f483b
```

```
$ sudo podman pod ps
POD ID         NAME      STATUS    CREATED         # OF CONTAINERS   INFRA ID
ae47ad61e0c5   demoweb Running   4 minutes ago 2              57267bfdbaec
560e4e44e0ae   demogen Running   4 minutes ago 2              92b74e53d6d2
2ff8097fcdc6   demodb    Running   4 minutes ago   2     eeaf676e115e
```

我们现在已经像以前一样重新创建了我们的容器和 pod，这将我们引向另一个测试用例。假设我们想通过添加两个额外的生成随机数的容器来增加数据库容器中生成的随机数的数量。我们现在可以这样做:

```
$ sudo podman run -dt --pod demogen quay.io/baude/demogen:latest python3 generatenums.py
a36adcc653566b130158738f06b1934bee28d0817b4fc1fd7b6d66296c989335
$ sudo podman run -dt --pod demogen quay.io/baude/demogen:latest python3 generatenums.py
8e2b28266595e8c2e46fc872ae3638985f0ce65d0c12b5a4c28359d2cc543ef7
```

```
$ sudo podman pod ps
POD ID         NAME          STATUS     CREATED         # OF CONTAINERS INFRA ID
ae47ad61e0c5   demoweb       Running   8 minutes ago    2               57267bfdbaec
560e4e44e0ae   demogen       Running   8 minutes ago    4               92b74e53d6d2
2ff8097fcdc6   demodb        Running   8 minutes ago    2               eeaf676e115e
```

如您所见，我们向`demogen` pod 添加了两个额外的容器。如果我们想在 Kubernetes 环境中测试这个，我们可以简单地重新运行`podman generate kube`并重新生成 Kubernetes YAML，然后在我们的 minikube 环境中重新部署这个描述。

## 总结

正如我们之前看到的生成的 YAML 所说，从波德曼生成库伯内特 YAML 的能力仍在开发中。具体来说，像 SELinux 这样的东西还没有被集成。因此，重放同一个 YAML 的能力也是如此。但是当前的功能应该开始为 Podman 用户提供运行、重新运行和保存他们的开发环境的方法，更重要的是，帮助他们提供在 Kubernetes 编排的环境中运行它们的能力。

## 关于 Podman 的更多信息，请访问 Red Hat 开发者博客

*   [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)
*   [使用 Podman 管理集装箱化系统服务](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/)
*   没有守护进程的容器:RHEL 7.6 和 RHEL 8 测试版中的 Podman 和 Buildah
*   [pod man——下一代 Linux 容器工具](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)
*   [pod man 简介(Red Hat Enterprise Linux 7.6 中的新功能)](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/)

要参与进来，看看波德曼是如何发展的，一定要查看一下[项目页面](https://github.com/containers/libpod)和[波德曼. io](https://podman.io/) 。

*Last updated: December 10, 2021*