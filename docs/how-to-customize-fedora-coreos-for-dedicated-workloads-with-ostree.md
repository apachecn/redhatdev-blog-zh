# 如何使用 OSTree 为专用工作负载定制 Fedora CoreOS

> 原文：<https://developers.redhat.com/blog/2020/03/12/how-to-customize-fedora-coreos-for-dedicated-workloads-with-ostree>

在本系列第一部分的[中，我介绍了](https://developers.redhat.com/blog/2020/03/10/how-to-run-containerized-workloads-securely-and-at-scale-with-fedora-coreos/) [Fedora CoreOS](https://getfedora.org/en/coreos/) (和 [Red Hat CoreOS](http://coreos.com/) )并解释了为什么它的不可变和原子性质对运行容器很重要。然后，我带您完成了获取 Fedora CoreOS、创建点火文件、引导 Fedora CoreOS、登录和运行测试容器的过程。在本文中，我将带您定制 Fedora CoreOS，并利用其不可变的原子特性。

## 扩展点火文件

在第一部分中，我们看到了一个基本示例，它包含一个最小的点火文件，我们从一个 FCC 文件生成该文件，然后注入一个公共 SSH 密钥。我们通过添加更多的逻辑来扩展这个例子；例如，创建一个`systemd`单元。既然我们正在开发一个容器优化系统，为什么不与 Podman 一起创建一个`systemd`单元呢？由于 Podman 的无后台特性，我们可以在一个 systemd 单元中运行、启动或停止我们的容器，并轻松管理它们的启动顺序。

为此，我扩展了以前的 FCC 文件:

```
variant: fcos
version: 1.0.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAAB3Nza...

systemd:
  units:
    - name: hello.service
      enabled: true
      contents: |
        [Unit]
        Description=A hello world unit!
        After=network-online.target
        Wants=network-online.target
        [Service]
        Type=forking
        KillMode=none
        Restart=on-failure
        RemainAfterExit=yes
        ExecStartPre=podman pull quay.io/gbsalinetti/hello-server
        ExecStart=podman run -d --name hello-server -p 8080:8080 quay.io/gbsalinetti/hello-server
        ExecStop=podman stop -t 10 hello-server
        ExecStopPost=podman rm hello-server
        [Install]
        WantedBy=multi-user.target
```

这一次，我添加了一个简单的单元文件来启动一个自制的映像，该映像运行一个最小且短暂的 Go web 服务器，然后打印一条“Hello World”消息。例子[的源代码可以在这里找到](https://github.com/giannisalinetti/hello-server.git)。看一下文件的`systemd`部分和它的`units`子部分，它包含一个代表一个或多个单元文件的项目列表。在这个块中，我们可以根据需要定义任意多的单元，它们将在引导时创建和启动。

我们可以使用点火配置来管理存储和用户；创建文件、目录和`systemd`单元；并注入 ssh 密钥。带有点火语法示例的详细语法文档可在 fcct-config 文档中找到[。](https://docs.fedoraproject.org/en-US/fedora-coreos/fcct-config/)

定制 FCC 文件后，我们必须使用`fcct`工具生成点火文件:

```
$ fcct -input example-fcc-systemd.yaml -output example-ignition-systemd.json
```

## 测试新实例

我们准备使用`virt-install`命令将生成的点火文件应用到 FCOS 新实例:

```
$ sudo virt-install --connect qemu:///system \
-n fcos -r 2048 --os-variant=fedora31 --import \
--graphics=none \
--disk size=10,backing_store=/home/gbsalinetti/Labs/fedora-coreos/fedora-coreos-31.20200118.3.0-qemu.x86_64.qcow2 \
--qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/home/gbsalinetti/Labs/fedora-coreos/example-ignition-systemd.ign"
```

在引导过程结束时，让我们登录并检查容器是否正在运行(在`ssh`命令中相应地更新您的 IP 地址):

```
$ ssh core@192.168.122.11
Fedora CoreOS 31.20200118.3.0
Tracker: https://github.com/coreos/fedora-coreos-tracker

Last login: Fri Feb  7 23:22:31 2020 from 192.168.122.1
[core@localhost ~]$ systemctl status hello.service
● hello.service - A hello world unit!
   Loaded: loaded (/etc/systemd/system/hello.service; enabled; vendor preset: enabled)
   Active: active (exited) since Fri 2020-02-07 23:18:39 UTC; 12min ago
  Process: 2055 ExecStartPre=/usr/bin/podman pull quay.io/gbsalinetti/hello-server (code=exited, status=0/SUCCESS)
  Process: 2112 ExecStart=/usr/bin/podman run -d --name hello-server -p 8080:8080 quay.io/gbsalinetti/hello-server (code=exited, status=0/SUCCESS)
 Main PID: 2112 (code=exited, status=0/SUCCESS)

Feb 07 23:18:17 localhost podman[2055]: Writing manifest to image destination
Feb 07 23:18:17 localhost podman[2055]: Storing signatures
Feb 07 23:18:38 localhost podman[2055]: 2020-02-07 23:18:38.671593577 +0000 UTC m=+47.966065770 image pull  
Feb 07 23:18:38 localhost podman[2055]: 146c93bfc4df81797068fdc26ee396348ba8c83a2d21b2d7dffc242dcdf38adb
Feb 07 23:18:38 localhost systemd[1]: Started A hello world unit!.
Feb 07 23:18:39 localhost podman[2112]: 2020-02-07 23:18:39.020399261 +0000 UTC m=+0.271239416 container create 2abf8d30360c03aead01092bbd8a8a51182a603911aac>
Feb 07 23:18:39 localhost podman[2112]: 2020-02-07 23:18:39.801631894 +0000 UTC m=+1.052472079 container init 2abf8d30360c03aead01092bbd8a8a51182a603911aac9f>
Feb 07 23:18:39 localhost podman[2112]: 2020-02-07 23:18:39.845449198 +0000 UTC m=+1.096289478 container start 2abf8d30360c03aead01092bbd8a8a51182a603911aac9>
Feb 07 23:18:39 localhost podman[2112]: 2abf8d30360c03aead01092bbd8a8a51182a603911aac9f8b4f5a465f0360f05
Feb 07 23:18:39 localhost systemd[1]: hello.service: Succeeded.
```

我们已经推出了一个容器作为一次性的波德曼命令。在本例中，命令以 0/SUCCESS 状态退出，容器成功启动。

为了检查正在运行的容器，我们可以使用`podman ps`命令:

```
[core@localhost ~]$ sudo podman ps
CONTAINER ID  IMAGE                                    COMMAND       CREATED         STATUS             PORTS                   NAMES
2abf8d30360c  quay.io/gbsalinetti/hello-server:latest  hello-server  14 minutes ago  Up 14 minutes ago  0.0.0.0:8080->8080/tcp  hello-server
```

为什么要用`sudo`？我们在`systemd`下启动了 Podman，因此该命令是以 root 用户权限执行的。这个结果就是*不是无根容器*，也叫*有根容器*。(无根容器以标准用户权限执行，有根容器以根权限执行。)

## `rpm-ostree`工具

现在我们已经学习了点火文件，让我们从 RPM 包安装开始学习如何处理系统更改，并学习在 FCOS 如何处理包安装。如果我们试图在 Fedora CoreOS 中启动`yum`命令，我们会收到“命令未找到”的错误。我们运行`dnf`得到同样的结果。相反，在这种体系结构中管理包的实用程序必须在原子文件系统管理库之上包装基于 RPM 的包管理。每个包更改都必须提交到文件系统。

我们之前已经提到过`rpm-ostree`工具。让我们用它来对基本操作系统映像进行持久化修改，然后安装[`buildah`包](https://buildah.io/)。Buildah 是管理开放容器倡议(OCI)映像构建的一个很好的工具，因为它复制了 docker 文件中的所有指令，允许我们在不需要 root 权限的情况下构建有或没有 docker 文件的映像。

### 安装新软件包

安装前简单回顾一下:FCOS 没有安装`dnf`或`yum`工具，`rpm-ostree`(构建在`libostree`库之上)是默认的包管理器。更改在内部提交，系统重新启动以应用新层。

让我们在安装之前检查一下系统的状态:

```
[core@localhost ~]$ rpm-ostree status
State: idle
AutomaticUpdates: disabled
Deployments:
● ostree://fedora:fedora/x86_64/coreos/stable
                   Version: 31.20200118.3.0 (2020-01-28T16:10:53Z)
                    Commit: 093f7da6ffa161ae1648a05be9c55f758258ab97b55c628bea5259f6ac6e370e
              GPGSignature: Valid signature by 7D22D5867F2A4236474BF7B850CB390B3C3359C4
```

我们在这里只能看到一个层，具有特定的提交 ID。`rpm-ostree status --json`命令可用于打印扩展状态。

现在，让我们用`rpm-ostree install`命令安装`buildah`包:

```
$ sudo rpm-ostree install buildah
```

让我们再次检查`rpm-ostree status`输出:

```
[core@localhost ~]$ rpm-ostree status
State: idle
AutomaticUpdates: disabled
Deployments:
  ostree://fedora:fedora/x86_64/coreos/stable
                   Version: 31.20200118.3.0 (2020-01-28T16:10:53Z)
                BaseCommit: 093f7da6ffa161ae1648a05be9c55f758258ab97b55c628bea5259f6ac6e370e
              GPGSignature: Valid signature by 7D22D5867F2A4236474BF7B850CB390B3C3359C4
                      Diff: 1 added
           LayeredPackages: buildah

● ostree://fedora:fedora/x86_64/coreos/stable
                   Version: 31.20200118.3.0 (2020-01-28T16:10:53Z)
                    Commit: 093f7da6ffa161ae1648a05be9c55f758258ab97b55c628bea5259f6ac6e370e
              GPGSignature: Valid signature by 7D22D5867F2A4236474BF7B850CB390B3C3359C4
```

新的提交增加了一个新的层，分层的包列表显示了我们之前安装的 Buildah 包。然后，我们可以根据自己的需要分层包装；例如，`tcpdump`:

```
[core@localhost ~]$ sudo rpm-ostree install tcpdump
```

再次运行`rpm-ostree status`命令时:

```
[core@localhost ~]$ rpm-ostree status
State: idle
AutomaticUpdates: disabled
Deployments:
  ostree://fedora:fedora/x86_64/coreos/stable
                   Version: 31.20200118.3.0 (2020-01-28T16:10:53Z)
                BaseCommit: 093f7da6ffa161ae1648a05be9c55f758258ab97b55c628bea5259f6ac6e370e
              GPGSignature: Valid signature by 7D22D5867F2A4236474BF7B850CB390B3C3359C4
                      Diff: 2 added
           LayeredPackages: buildah tcpdump

● ostree://fedora:fedora/x86_64/coreos/stable
                   Version: 31.20200118.3.0 (2020-01-28T16:10:53Z)
                    Commit: 093f7da6ffa161ae1648a05be9c55f758258ab97b55c628bea5259f6ac6e370e
              GPGSignature: Valid signature by 7D22D5867F2A4236474BF7B850CB390B3C3359C4
```

新的提交有两个分层的包。现在让我们启动`buildah`:

```
[core@localhost ~]$ buildah --help
-bash: buildah: command not found
```

哎哟，看起来二进制文件还没有安装。同样，这个响应是正确的，因为系统仍然运行在基础层之上。要解决这个问题并查看分层的包，我们需要重新启动系统，以便它启动到新的文件系统层:

```
[core@localhost ~]$ sudo systemctl reboot
```

现在我们可以享受之前安装的分层包了，因为它们在重启后出现在文件系统中。

## 包管理如何工作

当安装包时，我们并没有真正创建一个新的提交。相反，我们在当前系统提交的基础上用一个新的部署对包进行分层。那些分层的包将会在更新和重置时保持不变。换句话说，`rpm-ostree`融合了图像层和包管理的精华。

### 回滚更改

原子系统的伟大之处在于支持*原子回滚*。当我们在更新后达到不稳定的配置，并需要立即返回到工作稳定的系统时，此功能非常有用:

```
[core@localhost ~]$ sudo rpm-ostree rollback 
Moving '093f7da6ffa161ae1648a05be9c55f758258ab97b55c628bea5259f6ac6e370e.0' to be first deployment
Transaction complete; bootconfig swap: no; deployment count change: 0
Removed:
  buildah-1.12.0-2.fc31.x86_64
  tcpdump-14:4.9.3-1.fc31.x86_64
Run "systemctl reboot" to start a reboot
```

同样，需要重新启动才能使用回滚的层引导系统。

### 卸载

如果我们需要卸载单个包而不回滚所有内容，我们可以使用`rpm-ostree uninstall`命令:

```
[core@localhost ~]$ sudo rpm-ostree uninstall buildah
Staging deployment... done
Removed:
  buildah-1.12.0-2.fc31.x86_64
Run "systemctl reboot" to start a reboot
```

重启后，该软件包将不再可用。

## OSTree 的更多深入信息

`libostree`库提供了一个 API 来操作
不可变文件系统上的原子事务，方法是将它们作为整个树来管理。`ostree`命令是用于管理这些变更的默认工具。有许多语言绑定有助于创建我们自己的实现；例如， [`ostree-go`](https://github.com/ostreedev/ostree-go) 就是为 Golang 绑定的语言。

### 列出分行

我们可以使用`ostree refs`命令看到两个文件系统层之间的差异。首先，我们需要定位提交层的不同的*引用*:

```
[core@localhost ~]$ ostree refs
rpmostree/pkg/buildah/1.12.0-2.fc31.x86__64
rpmostree/base/0
ostree/0/1/1
rpmostree/pkg/tcpdump/14_3A4.9.3-1.fc31.x86__64
fedora:fedora/x86_64/coreos/stable
ostree/0/1/0
```

将 refs 想象成不同的分支，就像在 Git 存储库中一样。注意由`rpm-ostree`创建的分支，从模式`rpmostree/pkg/`开始。

### 检查`refs`内容

为了检查分支中发生的变化，我们可以使用`ostree ls`命令。例如，要查看由`buildah`包更改的文件和目录:

```
[core@localhost ~]$ ostree ls -R rpmostree/pkg/buildah/1.12.0-2.fc31.x86__64
d00755 0 0      0 /
d00755 0 0      0 /usr
d00755 0 0      0 /usr/bin
-00755 0 0 37474136 /usr/bin/buildah
d00755 0 0      0 /usr/lib
d00755 0 0      0 /usr/lib/.build-id
d00755 0 0      0 /usr/lib/.build-id/db
l00777 0 0      0 /usr/lib/.build-id/db/7c450ca346e503747a530a7185d9c16dc9a132 -> ../../../../usr/bin/buildah
d00755 0 0      0 /usr/share
d00755 0 0      0 /usr/share/bash-completion
d00755 0 0      0 /usr/share/bash-completion/completions
-00644 0 0  22816 /usr/share/bash-completion/completions/buildah
d00755 0 0      0 /usr/share/doc
d00755 0 0      0 /usr/share/doc/buildah
-00644 0 0   8450 /usr/share/doc/buildah/README.md
d00755 0 0      0 /usr/share/licenses
d00755 0 0      0 /usr/share/licenses/buildah
-00644 0 0  11357 /usr/share/licenses/buildah/LICENSE
d00755 0 0      0 /usr/share/man
d00755 0 0      0 /usr/share/man/man1
-00644 0 0    719 /usr/share/man/man1/buildah-add.1.gz
-00644 0 0  10290 /usr/share/man/man1/buildah-bud.1.gz
-00644 0 0   2264 /usr/share/man/man1/buildah-commit.1.gz
-00644 0 0   2426 /usr/share/man/man1/buildah-config.1.gz
-00644 0 0   1359 /usr/share/man/man1/buildah-containers.1.gz
-00644 0 0    662 /usr/share/man/man1/buildah-copy.1.gz
-00644 0 0   8654 /usr/share/man/man1/buildah-from.1.gz
-00644 0 0   1691 /usr/share/man/man1/buildah-images.1.gz
-00644 0 0    891 /usr/share/man/man1/buildah-info.1.gz
-00644 0 0    635 /usr/share/man/man1/buildah-inspect.1.gz
-00644 0 0   1029 /usr/share/man/man1/buildah-login.1.gz
-00644 0 0    637 /usr/share/man/man1/buildah-logout.1.gz
-00644 0 0   1051 /usr/share/man/man1/buildah-manifest-add.1.gz
-00644 0 0    856 /usr/share/man/man1/buildah-manifest-annotate.1.gz
-00644 0 0    615 /usr/share/man/man1/buildah-manifest-create.1.gz
-00644 0 0    334 /usr/share/man/man1/buildah-manifest-inspect.1.gz
-00644 0 0    907 /usr/share/man/man1/buildah-manifest-push.1.gz
-00644 0 0    459 /usr/share/man/man1/buildah-manifest-remove.1.gz
-00644 0 0    608 /usr/share/man/man1/buildah-manifest.1.gz
-00644 0 0   1072 /usr/share/man/man1/buildah-mount.1.gz
-00644 0 0   2144 /usr/share/man/man1/buildah-pull.1.gz
-00644 0 0   2440 /usr/share/man/man1/buildah-push.1.gz
-00644 0 0    200 /usr/share/man/man1/buildah-rename.1.gz
-00644 0 0    363 /usr/share/man/man1/buildah-rm.1.gz
-00644 0 0    905 /usr/share/man/man1/buildah-rmi.1.gz
-00644 0 0   4268 /usr/share/man/man1/buildah-run.1.gz
-00644 0 0    225 /usr/share/man/man1/buildah-tag.1.gz
-00644 0 0    298 /usr/share/man/man1/buildah-umount.1.gz
-00644 0 0   1111 /usr/share/man/man1/buildah-unshare.1.gz
-00644 0 0    312 /usr/share/man/man1/buildah-version.1.gz
-00644 0 0   2539 /usr/share/man/man1/buildah.1.gz
```

也可以列出分支中的单个目录:

```
[core@localhost ~]$ ostree ls -R rpmostree/pkg/buildah/1.12.0-2.fc31.x86__64 /usr/bin
d00755 0 0      0 /usr/bin
-00755 0 0 37830616 /usr/bin/buildah

```

我们还可以用`ostree checkout`命令检查到外部目录的分支。以下示例适用于`buildah` ref:

```
[core@localhost ~]$ sudo ostree checkout rpmostree/pkg/buildah/1.12.0-2.fc31.x86__64 /tmp/buildah_checkout
```

运行上述命令后，`/tmp/buildah_checkout`文件夹将包含从`rpmostree/pkg/buildah/1.12.0-2.fc31.x86__64`分支检出的所有文件。这是调试和故障诊断的一个重要功能。

## 重置基础

现在我们已经引入了`ostree`命令，我们可以展示一个惊人的特性:*重置*。通过改变基础，我们可以转移到不同的分支，并从根本上修改我们的文件系统树。

有没有想过如何从稳定版本切换到测试版本，然后在不重新安装的情况下切换到原始版本？有了重置基础，这是可能的！

`rpm-ostree rebase`命令可以将一个分支作为参数，并将整个系统移动到该分支。如果我们回忆一下`ostree refs`命令的输出，有下面一行:`fedora:fedora/x86_64/coreos/stable`。这个输出意味着我们在 Fedora CoreOS 的`stable`分支上。如果我们想切换到`testing`分行，我们只需重新设定基础，如下例所示。

```
[core@localhost ~]$ sudo rpm-ostree rebase -b fedora:fedora/x86_64/coreos/testing
⠒ Receiving objects: 99% (5030/5031) 627.5 kB/s 151.2 MB 
Receiving objects: 99% (5030/5031) 627.5 kB/s 151.2 MB... done
Staging deployment... done
Upgraded:
  NetworkManager 1:1.20.8-1.fc31 -> 1:1.20.10-1.fc31
  NetworkManager-libnm 1:1.20.8-1.fc31 -> 1:1.20.10-1.fc31
  e2fsprogs 1.45.3-1.fc31 -> 1.45.5-1.fc31
  e2fsprogs-libs 1.45.3-1.fc31 -> 1.45.5-1.fc31
  fuse-overlayfs 0.7.3-2.fc31 -> 0.7.5-2.fc31
  glibc 2.30-8.fc31 -> 2.30-10.fc31
  glibc-all-langpacks 2.30-8.fc31 -> 2.30-10.fc31
  glibc-common 2.30-8.fc31 -> 2.30-10.fc31
  kernel 5.4.10-200.fc31 -> 5.4.13-201.fc31
  kernel-core 5.4.10-200.fc31 -> 5.4.13-201.fc31
  kernel-modules 5.4.10-200.fc31 -> 5.4.13-201.fc31
  libcom_err 1.45.3-1.fc31 -> 1.45.5-1.fc31
  libsolv 0.7.10-1.fc31 -> 0.7.11-1.fc31
  libss 1.45.3-1.fc31 -> 1.45.5-1.fc31
  pcre2 10.33-16.fc31 -> 10.34-4.fc31
  selinux-policy 3.14.4-43.fc31 -> 3.14.4-44.fc31
  selinux-policy-targeted 3.14.4-43.fc31 -> 3.14.4-44.fc31
  socat 1.7.3.3-2.fc31 -> 1.7.3.4-1.fc31
  toolbox 0.0.17-1.fc31 -> 0.0.18-1.fc31
  whois-nls 5.5.4-1.fc31 -> 5.5.4-2.fc31
  zchunk-libs 1.1.4-1.fc31 -> 1.1.5-1.fc31
Removed:
  buildah-1.12.0-2.fc31.x86_64
Run "systemctl reboot" to start a reboot
```

注意升级到从`stable`转移到`testing`分支的包列表。现在，`ostree refs`产量略有不同，用`testing`分支代替`stable`:

```
[core@localhost ~]$ ostree refs
rpmostree/pkg/buildah/1.12.0-2.fc31.x86__64
rpmostree/base/0
ostree/0/1/1
rpmostree/pkg/tcpdump/14_3A4.9.3-1.fc31.x86__64
fedora:fedora/x86_64/coreos/testing
ostree/0/1/0
```

这个[发布流链接](https://github.com/coreos/fedora-coreos-tracker/blob/master/Design.md#release-streams)包含了关于如何管理分支的更详细的讨论。

## 文件系统分析

很容易注意到有些目录是不可写的；例如，当试图写入`/usr`时，我们得到一个只读文件系统错误。当系统启动时，只有某些部分(如`/var`)是可写的。那么，`ostree`是如何管理文件的呢？

`ostree`架构设计声明系统只读内容保存在`/usr`目录中。`/var`目录在所有系统部署中共享，可由进程写入，每个系统部署都有一个`/etc`目录。当系统改变或升级时，以前在`/etc`目录中的修改将与新部署中的副本合并。

所有这些更改都存储在`/ostree/repo`的 OSTree 存储库中，这是一个到`/sysroot/ostree/repo`的符号链接。该目录存储所有系统部署。将`/sysroot/ostree/repo`想象成 Git 存储库中的`.git`目录。

要检查当前状态并识别活动部署 UUID，请使用以下命令:

```
[core@localhost ~]$ sudo ostree admin status
* fedora-coreos 4ea6beed22d0adc4599452de85820f6e157ac1750e688d062bfedc765b193505.0
    Version: 31.20200210.3.0
    origin refspec: fedora:fedora/x86_64/coreos/stable
```

如果我们检查新安装的系统中的`/ostree`文件夹，我们会发现一个包含引导文件系统树的精确匹配:

```
[core@localhost ~]$ ls -al /ostree/boot.1/fedora-coreos/19190477fad0e60d605a623b86e06bb92aa318b6b79f78696b06f68f262ad5d6/0/
total 8
drwxr-xr-x. 12 root root 253 Feb 24 16:52 .
drwxrwxr-x. 3 root root 161 Feb 24 16:52 ..
lrwxrwxrwx. 2 root root 7 Feb 24 16:51 bin -> usr/bin
drwxr-xr-x. 2 root root 6 Jan 1 1970 boot
drwxr-xr-x. 2 root root 6 Jan 1 1970 dev
drwxr-xr-x. 77 root root 4096 Mar 11 08:54 etc
lrwxrwxrwx. 2 root root 8 Feb 24 16:51 home -> var/home
lrwxrwxrwx. 2 root root 7 Feb 24 16:51 lib -> usr/lib
lrwxrwxrwx. 2 root root 9 Feb 24 16:51 lib64 -> usr/lib64
lrwxrwxrwx. 2 root root 9 Feb 24 16:51 media -> run/media
lrwxrwxrwx. 2 root root 7 Feb 24 16:51 mnt -> var/mnt
lrwxrwxrwx. 2 root root 7 Feb 24 16:51 opt -> var/opt
lrwxrwxrwx. 2 root root 14 Feb 24 16:51 ostree -> sysroot/ostree
drwxr-xr-x. 2 root root 6 Jan 1 1970 proc
lrwxrwxrwx. 2 root root 12 Feb 24 16:51 root -> var/roothome
drwxr-xr-x. 2 root root 6 Jan 1 1970 run
lrwxrwxrwx. 2 root root 8 Feb 24 16:51 sbin -> usr/sbin
lrwxrwxrwx. 2 root root 7 Feb 24 16:51 srv -> var/srv
drwxr-xr-x. 2 root root 6 Jan 1 1970 sys
drwxr-xr-x. 2 root root 6 Jan 1 1970 sysroot
drwxrwxrwt. 2 root root 6 Mar 11 08:37 tmp
drwxr-xr-x. 12 root root 155 Jan 1 1970 usr
drwxr-xr-x. 4 root root 28 Mar 11 08:37 var
```

上面的目录代表被引导的部署，实际上是一个到`/ostree/deploy/fedora-cores/deploy/`下的 ref 的符号链接。注意，目录的名称将与`ostree admin status`命令输出的 UUID 相匹配。

在系统引导时，文件系统根目录中的一些目录被创建为到活动部署树的硬链接。由于硬链接指向文件系统中的同一个 inode 编号，让我们交叉检查一下`/usr`文件夹和引导部署中的 inode 编号:

```
[core@localhost ~]$ ls -alid /usr
3218784 drwxr-xr-x. 12 root root 155 Jan 1 1970 /usr
[core@localhost ~]$ ls -alid /sysroot/ostree/boot.1/fedora-coreos/19190477fad0e60d605a623b86e06bb92aa318b6b79f78696b06f68f262ad5d6/0/usr/
3218784 drwxr-xr-x. 12 root root 155 Jan 1 1970 /sysroot/ostree/boot.1/fedora-coreos/19190477fad0e60d605a623b86e06bb92aa318b6b79f78696b06f68f262ad5d6/0/usr/
```

正如所料，两个目录的索引节点编号 3218784 是相同的，这表明文件系统根目录下的内容是由活动部署组成的。

我们用`rpm-ostree`安装 Buildah 包的时候发生了什么？重新启动后，将出现一个新的部署:

```
[core@localhost ~]$ sudo ostree admin status
* fedora-coreos ee678bde3c15d8cae34515e84e2b4432ba3d8c9619ca92c319b576a13029481d.0
Version: 31.20200210.3.0
origin: <unknown origin type>
fedora-coreos 4ea6beed22d0adc4599452de85820f6e157ac1750e688d062bfedc765b193505.0 (rollback)
Version: 31.20200210.3.0
origin refspec: fedora:fedora/x86_64/coreos/stable
```

请注意当前活动部署旁边的星号。我们希望在`/ostree/deploy/fedora-cores/deploy`下找到它和旧的:

```
[core@localhost ~]$ ls -al /ostree/deploy/fedora-coreos/deploy/
total 12
drwxrwxr-x. 4 root root 4096 Mar 11 10:18 .
drwxrwxr-x. 4 root root 31 Feb 24 16:52 ..
drwxr-xr-x. 12 root root 253 Feb 24 16:52 4ea6beed22d0adc4599452de85820f6e157ac1750e688d062bfedc765b193505.0
-rw-r--r--. 1 root root 52 Feb 24 16:52 4ea6beed22d0adc4599452de85820f6e157ac1750e688d062bfedc765b193505.0.origin
drwxr-xr-x. 12 root root 253 Mar 11 10:01 ee678bde3c15d8cae34515e84e2b4432ba3d8c9619ca92c319b576a13029481d.0
-rw-r--r--. 1 root root 87 Mar 11 10:18 ee678bde3c15d8cae34515e84e2b4432ba3d8c9619ca92c319b576a13029481d.0.origin
```

`/ostree/deploy/fedora-cores`目录也称为 *stateroot* 或 *osname* ，它包含部署及其所有相关的提交。在每个 stateroot 目录中，只有一个`var`目录使用`systemd`挂载单元挂载在`/var`下(由文件`/run/systemd/generator/var.mount`表示)。

让我们通过查看`/ostree/repo`文件夹来完成对文件系统的深入研究。这是我们可以找到与 Git 最相似的地方:

```
[core@localhost repo]$ ls -al /ostree/repo/
total 16
drwxrwxr-x. 7 root root 102 Mar 11 10:18 .
drwxrwxr-x. 5 root root 62 Mar 11 10:18 ..
-rw-r--r--. 1 root root 73 Feb 24 16:52 config
drwxr-xr-x. 3 root root 23 Mar 11 09:59 extensions
-rw-------. 1 root root 0 Feb 24 16:51 .lock
drwxr-xr-x. 258 root root 8192 Feb 24 16:52 objects
drwxr-xr-x. 5 root root 49 Feb 24 16:51 refs
drwxr-xr-x. 2 root root 6 Feb 24 16:52 state
drwxr-xr-x. 3 root root 19 Mar 11 10:18 tmp
```

注意`refs`和`objects`文件夹，它们分别存储分支信息和版本控制对象。这里，我们在一个典型的 Git 库的`.git`文件夹中有一个精确的匹配。

## 管理系统升级

现在应该很清楚了，`rpm-ostree`工作在`ostree`库之上，并且用原子方法提供包管理。系统升级也是原子性的。升级系统只是在现有文件系统的基础上增加一个新的提交，我们可以用`rpm-ostree upgrade`命令轻松完成:

```
[core@localhost ~]$ sudo rpm-ostree upgrade -r
```

**注意:**`-r`标志告诉`rpm-ostree`升级完成后自动重启。

尽管允许用户手动升级系统，但 FCOS 提供了一项名为 [Zincati](https://github.com/coreos/zincati) 的专门服务来管理系统升级。Zincati 是一个执行定期升级检查并应用它们的代理。我们可以使用`systemctl`命令检查我们的 Zincati 服务状态:

```
[core@localhost ~]$ sudo systemctl status zincati
● zincati.service - Zincati Update Agent
   Loaded: loaded (/usr/lib/systemd/system/zincati.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-02-07 11:36:33 UTC; 3min 30s ago
     Docs: https://github.com/coreos/zincati
 Main PID: 707 (zincati)
    Tasks: 2 (limit: 2297)
   Memory: 17.5M
   CGroup: /system.slice/zincati.service
           └─707 /usr/libexec/zincati agent -v

Feb 07 11:36:33 localhost systemd[1]: Started Zincati Update Agent.
Feb 07 11:36:33 localhost zincati[707]: [INFO ] starting update agent (zincati 0.0.6)
Feb 07 11:36:39 localhost zincati[707]: [INFO ] Cincinnati service: https://updates.coreos.stg.fedoraproject.org
Feb 07 11:36:39 localhost zincati[707]: [INFO ] agent running on node '20d1f6332922438d8a8edede3fbe6251', in update group 'default'
Feb 07 11:36:39 localhost zincati[707]: [INFO ] initialization complete, auto-updates logic enabled
```

Zincati 行为可以自定义。默认配置已经安装在`/usr/lib/zincati/config.d/`下，而用户可以在`/etc/zincati/configs.d/`中应用自定义配置并覆盖默认配置。

## OpenShift 4 和机器配置操作符

开发者社区正在努力集成 OKD 4 和 Fedora CoreOS，我们正在等待 OKD 4 的第一个稳定版本。

如今，所有的[红帽 OpenShift 容器平台 4](https://developers.redhat.com/products/openshift) (RHOCP)，都运行在红帽 CoreOS (RHCOS)节点之上。了解 RHCOS 系统的管理方式非常有用。先来个挑衅。在 RHOCP 4 中，任何系统管理员都不应该通过 SSH 访问 RHCOS 或 Fedora CoreOS 节点来进行更改。

谁负责节点管理、升级，谁应用点火配置？在 OpenShift 中，所有这些任务都由[机器配置操作员](https://github.com/openshift/machine-config-operator) (MCO)管理，这在上一篇文章中已经提到过。

MCO 是一个核心操作器，它在 OpenShift 集群中产生不同的组件:

*   `machine-config-server` (MCS)通过 HTTPS 向节点提供点火文件。
*   `machine-config-controller` (MCC)协调机器升级到由`MachineConfig`对象定义的所需配置。
*   `machine-config-daemon` (MCD)作为 DaemonSet 在每个节点上运行，应用机器配置并根据请求的配置验证机器的状态。

MCD 使用 CoreOS 技术执行提供的点火文件中定义的配置。有关`machine-config-daemon`、[的更多细节，请阅读本文档](https://github.com/openshift/machine-config-operator/blob/master/docs/MachineConfigDaemon.md)。

MCD 使用 Podman 来管理系统升级，以获取和安装系统映像，并使用`rpm-ostree rebase`将 RHCOS 节点重新设置到已安装容器的文件系统树中。换句话说，OCI 映像用于将整个升级的文件系统传输到节点。

由 MCD 处理的`MachineConfig`对象只不过是嵌入点火配置的 OpenShift 资源。MachineConfigs 被分配给一个`MachineConfigPools`，并应用于属于该池的所有机器。池与群集中的节点角色直接匹配，默认情况下，我们在 OCP 4、`master`和`worker`中只有 2 个 MachineConfigPools，但是可以添加反映特定角色的自定义池，例如基础架构节点或 HPC 节点。

Machine Config 操作符及其管理的组件是基于不可变的原子方法创建的，在 Red Hat CoreOS 之上实现，以便自动化节点上的 day2 操作，并向客户交付一个 *NoOps* 容器平台。

MCO 架构为混合云环境带来了巨大的价值，在混合云环境中，基础架构自动化几乎是管理复杂场景的必备条件。

## 结论

对 Fedora CoreOS 和不可变系统特性的探索之旅已经结束，但这两篇文章只是开始。不可变的基础设施是下一件大事，不仅仅是在容器化的工作负载中。我认为，有了正确的工具，他们可以在传统、裸机和本地场景中改变游戏规则。

通过重建而不是更改可靠的系统，并使用类似 Git 的提交、分支和回滚方法来管理映像，我们可以真正拥抱基础设施即代码的文化。这种方法向我们展示了更易维护、可持续和稳定的系统。

*Last updated: June 29, 2020*