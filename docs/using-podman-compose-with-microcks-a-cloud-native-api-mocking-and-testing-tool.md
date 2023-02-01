# 使用 Podman Compose with Microcks:一个云原生 API 模拟和测试工具

> 原文：<https://developers.redhat.com/blog/2021/04/22/using-podman-compose-with-microcks-a-cloud-native-api-mocking-and-testing-tool>

[Microcks](https://microcks.io/) 是一个云原生 API 模拟和测试工具。它通过获取你的 OpenAPI 规范并从中生成实时模拟来帮助你覆盖你的 [API 的整个生命周期](/blog/2019/02/25/full-api-lifecycle-management-a-primer/)。它还可以断言您的 API 实现符合您的 OpenAPI 规范。你可以在各种各样的云原生平台上部署微型芯片，比如 [Kubernetes](/topics/kubernetes) 和 [Red Hat OpenShift](/products/openshift/overview) 。没有公司权限访问云原生平台的开发人员已经使用了 [Docker Compose](https://docs.docker.com/compose/) 。尽管 Docker 仍然是软件打包和安装的最流行的容器选项，但是 Podman 正在获得越来越多的关注。

波德曼被宣传为码头工人的替代者。倡导者给人的印象是，你可以发布`alias docker=podman`,你就可以走了。现实更加微妙，社区必须努力为波德曼获得适当的微芯片支持。

本文讨论了让微芯片与波德曼一起工作的障碍，以及我们为绕过它们而做出的设计决策。它包括一个在无根模式下使用 Podman 和 Microcks 的简单例子。

## 在微型芯片中支持波德曼

Podman 提出了一些设计障碍，Microcks 社区必须解决这些障碍。我们将讨论障碍和我们如何解决它们，以及这些决定对使用带有微芯片的 Podman 的开发者意味着什么。

### 有根还是无根？

Docker 要求以 root 用户身份运行一个守护进程，观察者长期以来一直批评这种做法不安全。Podman 采用了一种非常不同的架构:它根本不涉及任何守护进程，可以作为根用户(rootfull 模式)或普通用户(无根模式)运行。Microcks 支持有根或无根模式下的 Podman。

虽然无根模式看起来很吸引人，但它不是没有代价的。缺点包括:

*   容器没有 IP 地址和 DNS 别名。
*   端口重定向在用户空间中完成，而 rootfull 模式使用 [iptables](https://linux.die.net/man/8/iptables) ，速度更快。
*   覆盖存储是在用户空间用 [FUSE](https://cloud.google.com/storage/docs/gcs-fuse) 完成的，比传统的 [overlayFS](https://www.kernel.org/doc/html/latest/filesystems/overlayfs.html) 挂载要慢。

不过，除非您需要高性能或特定的网络设置，否则您可以使用无根模式。

### DNS 别名:仍然碍事

Microcks 需要适当的 DNS 别名才能正常工作。主要原因是 Microcks 使用 OpenID Connect 协议进行用户认证，这涉及到面向用户的交互和服务器到服务器的交互。

结合堆栈中其他软件(Keycloak、Docker、Podman)的怪癖和限制，这一需求解释了为什么在不修改 Docker Compose 配置的情况下运行微芯片总是具有挑战性。

为了解决这个问题，社区创建了一个 shell 脚本来发现当前机器的 IP 地址，并将该地址硬编码到`podman-compose`文件中。

要让 DNS 别名工作，您需要在默认的`podman`网络中启用`dnsname`插件。

### 建立工作关系网

Podman 中的 rootfull 模式带来了另一个挑战:默认情况下，容器只能通过它们的 IP 地址引用其他容器。该命令重新创建默认的`podman`网络并启用`dnsname`插件:

```
$ sudo podman network rm podman
$ sudo podman network create --subnet 10.88.0.0/16 podman

```

## 使用波德曼与微芯片组合

Podman 支持可能看起来不那么光彩，但是使用 Podman 的好处是值得努力的！

无根模式是让微芯片与 Podman Compose 一起工作的最简单、最安全的方式:

```
$ git clone https://github.com/microcks/microcks.git
$ cd microcks/install/podman-compose
$ ./run-microcks.sh
Running rootless containers...
Discovered host IP address: 192.168.3.102

Starting Microcks using podman-compose ...
------------------------------------------
Stop it with:  podman-compose -f microcks.yml --transform_policy=identity stop
Re-launch it with:  podman-compose -f microcks.yml --transform_policy=identity start
Clean everything with:  podman-compose -f microcks.yml --transform_policy=identity down
------------------------------------------
Go to https://localhost:8080 - first login with admin/123
Having issues? Check you have changed microcks.yml to your platform

using podman version: podman version 2.1.1
podman run [...]
```

Rootfull 模式要求您在默认的`podman`网络上启用`dnsname`插件，如前所述。然后，您只需用 sudo 运行这个脚本。

## 结论

让微芯片与波德曼一起工作并不特别困难。我们希望这一支持将帮助您开始在企业环境中使用微芯片。更多详情请阅读 microcks.io 上的[公告。](https://microcks.io/blog/podman-compose-support/)

*Last updated: October 14, 2022*