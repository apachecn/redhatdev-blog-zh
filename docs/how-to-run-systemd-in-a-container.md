# 如何在容器中运行系统

> 原文：<https://developers.redhat.com/blog/2019/04/24/how-to-run-systemd-in-a-container>

关于容器中的 [systemd](https://github.com/systemd/systemd) 我已经说了很久了。早在 2014 年，我就写过“[在 Docker 容器](https://developers.redhat.com/blog/2014/05/05/running-systemd-within-docker-container/)中运行 systemd。”几年后，我写了另一篇文章，“[在非特权容器](https://developers.redhat.com/blog/2016/09/13/running-systemd-in-a-non-privileged-container/)中运行 systemd”，解释事情并没有好转。在那篇文章中，我说，“可悲的是，两年后，如果你在谷歌上搜索 Docker systemd，这仍然是人们看到的文章——是时候更新了。”我还链接了一个关于【upstream Docker 和 upstream systemd 如何不妥协的演讲。在这篇文章中，我将看看已经取得的进展，以及 Podman 可以如何提供帮助。

在系统内部运行 systemd 有很多原因，例如:

1.  **多服务容器**—许多人希望将现有的多服务应用程序从虚拟机中取出，并在容器中运行。我们更希望他们将这些应用拆分成微服务，但有些人还不能或没有时间。因此，将它们作为 systemd 从单元文件中启动的服务来运行是有意义的。
2.  **Systemd 单元文件**—大多数在容器内运行的应用程序都是由在虚拟机或主机系统上运行的代码构建的。这些应用程序有一个为该应用程序编写的单元文件，并了解如何运行该应用程序。通过受支持的方法启动服务可能比破解自己的 init 服务更好。
3.  Systemd 是一个进程管理器(process manager)——它比其他任何工具都更好地处理了服务的管理，如获取、重启和关闭。

也就是说，也有很多理由不在容器中运行 systemd。主要的一点是 systemd/journald 控制容器的输出，而像 [Kubernetes](https://kubernetes.io/) 和 [OpenShift](https://www.openshift.com/) 这样的工具期望容器直接记录到 stdout 和 stderr。因此，如果您打算通过这样的 Orchestrator 来管理您的容器，那么您应该对使用基于 systemd 的容器三思而行。此外，Docker 和莫比的上游社区经常反对在容器中使用 systemd。

## 进入波德曼

我很高兴地说，情况有所好转。我在 Red Hat 的团队 container runtimes 决定开发我们自己的容器引擎，叫做 [Podman](https://github.com/containers/libpod) 。Podman 是一个容器引擎，具有与 Docker 相同的命令行界面(CLI)。几乎所有可以从 Docker 命令行运行的命令都可以用 Podman 来执行。我经常做一个现在叫做[的演讲，用搬运工](https://podman.io/talks/2018/10/01/talk-replace-docker-with-podman.html)代替 Docker，第一张幻灯片上写着`alias docker=podman`。

很多人都有。

然而，对于 Podman，我们并不敌视基于 systemd 的容器。Systemd 是这个星球上最流行的 Linux init 系统，不允许它在容器中正常运行会忽略成千上万的用户选择运行容器的方式。

Podman 知道 systemd 需要做什么才能在容器中运行。它需要在/run 和/tmp 安装 tmpfs 之类的东西。它喜欢打开“容器”环境，并希望能够写入 cgroup 目录中属于它的部分和/var/log/journald 目录。

当 Podman 启动一个运行 init 或 systemd 作为初始命令的容器时，Podman 会自动设置 tmpfs 和 Cgroups，以便 systemd 顺利启动。如果你想阻止 systemd 行为，你必须运行`--systemd=false`。请注意，systemd 行为仅在 Podman 看到要执行的命令是 systemd 或 init 时才会发生。

以下是手册页描述:

> 曼·波德曼跑
> 
> …
> 
> - systemd=true|false
> 
> 以 systemd 模式运行容器。默认值为 true。
> 
> 如果您在容器内部运行的命令是 systemd 或 init，那么 podman 将在以下目录中设置 tmpfs 挂载点:
> 
> /run、/run/lock、/tmp、/sys/fs/cgroup/systemd、/var/lib/journal
> 
> 它还会将默认停止信号设置为 SIGRTMIN+3。
> 
> 这允许 systemd 在一个封闭的容器中运行，无需任何修改。
> 
> 注意:在 SELinux 系统上，systemd 试图写入 cgroup 文件系统。默认情况下，拒绝容器写入 cgroup 文件系统。必须启用 container_manage_cgroup 布尔值，以便在 SELinux 独立系统上允许这样做。
> 
> setse tool-P container _ manage _ cgroup true

现在让我们看一个使用 Podman 在容器中运行 systemd 的 docker 文件:

```
# cat Dockerfile

FROM fedora

RUN dnf -y install httpd; dnf clean all; systemctl enable httpd

EXPOSE 80

CMD [ "/sbin/init" ]

```

就是这样。

构建容器

```
# podman build -t systemd .
```

告诉 SELinux 允许 systemd 操作它的 Cgroups 配置是可以的。

```
# setsebool -P container_manage_cgroup true
```

你会忘记这样做；我写这个博客的时候确实这么做了。幸运的是，你这样做一次，它将被设置为系统的生命周期。

现在运行容器。

```
# podman run -ti -p 80:80 systemd

systemd 239 running in system mode. (+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-hierarchy=hybrid)

Detected virtualization container-other.

Detected architecture x86-64.

Welcome to Fedora 29 (Container Image)!

Set hostname to <1b51b684bc99>.

Failed to install release agent, ignoring: Read-only file system

File /usr/lib/systemd/system/systemd-journald.service:26 configures an IP firewall (IPAddressDeny=any), but the local system does not support BPF/cgroup based firewalling.

Proceeding WITHOUT firewalling in effect! (This warning is only shown for the first loaded unit using IP firewalling.)

[  OK ] Listening on initctl Compatibility Named Pipe.

[  OK ] Listening on Journal Socket (/dev/log).

[  OK ] Started Forward Password Requests to Wall Directory Watch.

[  OK ] Started Dispatch Password Requests to Console Directory Watch.

[  OK ] Reached target Slices.

…

[  OK ] Started The Apache HTTP Server.

```

服务已经启动并运行。

```
$ curl localhost

<html  xml:lang="en" lang="en">

…

</html>

```

注意:不要用 Docker 尝试这种方法，你仍然需要经历重重困难才能让这样的容器在守护进程中运行。(您需要额外的字段和包，以便在 Docker 中无缝地工作，或者在特权容器中运行。[我之前的文章](https://developers.redhat.com/blog/2016/09/13/running-systemd-in-a-non-privileged-container/)更好地解释了这一点。)

## 关于 Podman 和 systemd 的其他很酷的特性

### 系统单元文件中的 podman 比 docker 工作得更好

当在引导时启动容器时，您可以简单地将 Podman 命令放入 systemd 单元文件中，systemd 将启动并监视服务。波德曼是一个标准的分叉和执行模型。这意味着容器进程是 Podman 进程的子进程，因此 systemd 可以轻松地监控这些进程。

Docker 是一个客户端服务模型，将 Docker CLI 放入一个单元文件是可能的。然而，当 Docker 客户机连接到 Docker 守护进程时，Docker 客户机就变成了另一个处理 stdin 和 stdout 的进程。Systemd 不知道 Docker 客户机和在 Docker 守护进程下运行的容器之间的这种关系，并且不能监视该模型中的服务。

### 系统套接字激活

当套接字被激活时，Podman 正常工作。因为 Podman 是一个 fork/exec 模型，所以它可以将连接的套接字向下传递给其子容器进程。由于客户端/服务器模式，Docker 无法做到这一点。

Podman [varlink](https://varlink.org/) ，一个 Podman 用于远程客户端与容器交互的服务，实际上是套接字激活的。用 Node.js 编写的 [cockpit-podman](https://github.com/cockpit-project/cockpit-podman) 包是 cockpit 项目的一部分，它允许人们通过 web 界面与 podman 容器进行交互。运行 cockpit-podman 的 web 守护进程向 systemd 正在监听的 varlink 套接字发送消息。Systemd 然后激活 Podman 程序来接收消息并开始管理容器。Systemd 套接字激活允许我们没有长时间运行的守护进程，仍然能够处理远程 API。

我们正在为 Podman 开发另一个客户端，名为 *podman-remote* ，它实现了相同的 Podman CLI，但调用 varlink 来启动容器。Podman-remote 可以在 SSH 会话上工作，允许我们安全地与不同机器上的容器进行交互。我们最终计划使用 podman-remote 来支持 MacOS 和 Windows 用户以及 Linux 用户。这将允许在 Mac 或 Windows 机器上的开发人员启动一个运行 Podman varlink 的 Linux VM，并感觉容器正在他们的本地机器上运行。

### SD _ 通知

Systemd 有能力阻止依赖于容器化服务启动的辅助服务启动。Podman 可以将 SD_NOTIFY 套接字传递给容器化的服务，这样它就可以在准备好开始服务请求时通知 systemd。由于客户机/服务器模型，Docker 也不能做到这一点。

## 未来的工作

我们计划添加一个`podman generate systemd CONTAINERID`，它将生成一个 systemd 单元文件来管理指定的容器。对于非特权容器，这应该在根或无根模式下工作。我甚至见过一个 PR 创建一个 systemd-nspawn OCI 兼容的运行时。

## 结论

在容器中运行 systemd 是合理的做法。最后，我们在 Podman 中有一个容器运行时，它并不反对完全运行 systemd，而是轻松地支持工作负载。

*Last updated: November 5, 2021*