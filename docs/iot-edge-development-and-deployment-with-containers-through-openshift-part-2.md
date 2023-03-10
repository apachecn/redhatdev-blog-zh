# 通过 OpenShift 使用容器进行物联网边缘开发和部署:第 2 部分

> 原文：<https://developers.redhat.com/blog/2019/02/05/iot-edge-development-and-deployment-with-containers-through-openshift-part-2>

在本系列第一部分的[中，我们看到了像](https://developers.redhat.com/blog/2019/01/31/iot-edge-development-and-deployment-with-containers-through-openshift-part-1) [Red Hat OpenShift](http://openshift.com/) 这样的平台即服务(PaaS)对于开发物联网边缘应用并将其分发到远程站点是多么有效，这要归功于[容器](https://developers.redhat.com/blog/category/containers/)和 [Red Hat Ansible Automation](https://www.redhat.com/en/technologies/management/ansible) 技术。

通常，我们认为物联网应用是专为能力有限的低功耗设备设计的。物联网设备可能使用不同的 CPU 架构或平台。由于这个原因，我们倾向于使用完全不同的技术来开发物联网应用程序，而不是运行在数据中心的服务。

在第二部分中，我们将探索一些技术，这些技术允许您在 x86_64 主机上为替代架构(如 ARM64)构建和测试容器。我们的目标是让您能够使用相同的语言、框架和开发工具来编写在数据中心或物联网边缘设备上运行的代码。在本文中，我将展示如何在 x86_64 主机上构建和运行 AArch64 容器映像，然后使用 Fedora 和 Podman 构建一个 RPI3 映像在物理硬件上运行。

在上一篇文章中，我假设物联网网关能够运行 x86_64 容器是一个先决条件，但不幸的是，由于目前市场上有各种类型的物联网网关，这并不是一个常见的用例。

我在 GitHub 上发现了一个非常有趣的项目，名为“multiarch ”,有多个存储库[。](https://github.com/multiarch)

该项目的目的是创建一种使用名为`binfmt_misc`的内置 Linux 内核特性的便捷方式，在维基百科中解释如下:“ *binfmt_misc 是 Linux 内核的一项功能，它允许识别任意可执行文件格式并将其传递给某些用户空间应用程序，如模拟器和虚拟机。它是内核中许多二进制格式处理程序中的一个，用于准备用户空间程序的运行。*

multiarch 项目背后的概念非常简单:想象一下启动一个特权容器，它能够与容器的主机进行交互，并使用`binfmt_misc`特性通知内核在`PATH`中的某个地方有其他二进制处理程序可用。

你在猜测训练者吗？它们只是 QEMU x86_64 可执行文件，能够运行特定的架构二进制文件:ARM、MIPS、Power 等。

请记住，QEMU 是一个通用的开源机器模拟器和虚拟化器，所以它是这项工作的合适人选。它还用于以接近本机的性能运行 KVM 虚拟机。

此时，您唯一要做的事情就是将正确的 QEMU 二进制文件放入具有不同架构的容器中。为什么一定要放在集装箱里？

QEMU 可执行文件必须放在容器中，因为当容器引擎试图执行一些其他的 ARCH 二进制文件时，它会在内核中触发`binfmt_misc`特性，然后这个特性会将执行重定向到我们指定的路径中的二进制文件。由于我们在容器的虚拟根文件系统中，QEMU 可执行文件必须驻留在刚刚运行的二进制文件的相同环境中。

功能激活非常简单。正如 multiarch 项目页面所述，我们只需要确保使用带有`--reset`选项的`multiarch/qemu-user-static:register`。

项目页面建议使用 Docker 来应用这个动作，当然，这个特性会在下一次机器重启时丢失，但是我们可以将其设置为一次性`systemd`服务。

对于本文，我们只是使用一个 Minishift 安装。出于这个原因，由于 Minishift 的基本操作系统缺乏持久性，我们将只在登录后运行 Docker 命令:

```
[alex@lenny ~]$ cdk-minishift ssh
[docker@minishift ~]$ docker run --rm --privileged multiarch/qemu-user-static:register --reset
Setting /usr/bin/qemu-alpha-static as binfmt interpreter for alpha
Setting /usr/bin/qemu-arm-static as binfmt interpreter for arm
Setting /usr/bin/qemu-armeb-static as binfmt interpreter for armeb
Setting /usr/bin/qemu-sparc32plus-static as binfmt interpreter for sparc32plus
Setting /usr/bin/qemu-ppc-static as binfmt interpreter for ppc
Setting /usr/bin/qemu-ppc64-static as binfmt interpreter for ppc64
Setting /usr/bin/qemu-ppc64le-static as binfmt interpreter for ppc64le
Setting /usr/bin/qemu-m68k-static as binfmt interpreter for m68k
Setting /usr/bin/qemu-mips-static as binfmt interpreter for mips
Setting /usr/bin/qemu-mipsel-static as binfmt interpreter for mipsel
Setting /usr/bin/qemu-mipsn32-static as binfmt interpreter for mipsn32
Setting /usr/bin/qemu-mipsn32el-static as binfmt interpreter for mipsn32el
Setting /usr/bin/qemu-mips64-static as binfmt interpreter for mips64
Setting /usr/bin/qemu-mips64el-static as binfmt interpreter for mips64el
Setting /usr/bin/qemu-sh4-static as binfmt interpreter for sh4
Setting /usr/bin/qemu-sh4eb-static as binfmt interpreter for sh4eb
Setting /usr/bin/qemu-s390x-static as binfmt interpreter for s390x
Setting /usr/bin/qemu-aarch64-static as binfmt interpreter for aarch64
Setting /usr/bin/qemu-aarch64_be-static as binfmt interpreter for aarch64_be
Setting /usr/bin/qemu-hppa-static as binfmt interpreter for hppa
Setting /usr/bin/qemu-riscv32-static as binfmt interpreter for riscv32
Setting /usr/bin/qemu-riscv64-static as binfmt interpreter for riscv64
Setting /usr/bin/qemu-xtensa-static as binfmt interpreter for xtensa
Setting /usr/bin/qemu-xtensaeb-static as binfmt interpreter for xtensaeb
Setting /usr/bin/qemu-microblaze-static as binfmt interpreter for microblaze
Setting /usr/bin/qemu-microblazeel-static as binfmt interpreter for microblazeel
```

正如您在前面的命令中看到的，我们刚刚为当前的 x86_64 主机注册了一组处理程序，这些处理程序将从主机内核接收针对不同架构的指令的特定请求。

我们现在已经准备好用 x86_64 之外的架构测试容器构建项目。

为此，我准备了一个简单的测试项目，用于构建一个 ARM 64 位容器映像，并将其与一个 Raspberry Pi 3 一起使用:它只是一个 web 服务器。

正如您将在[项目页面](https://github.com/alezzandro/test-arm-container)中看到的，git repo 仅包含一个 Dockerfile 文件:

```
FROM multiarch/debian-debootstrap:arm64-stretch-slim

RUN apt-get update
RUN apt-get install -y apache2
RUN sed -i 's/80/8080/g' /etc/apache2/ports.conf

EXPOSE 8080

CMD ["/usr/sbin/apache2ctl", "-DFOREGROUND"]
```

它从具有 ARM64 架构的 Debian 基础映像开始。它更新 APT repos 并安装 web 服务器。之后，它会替换默认监听端口，然后设置正确的`init`命令。

我打赌你在问，“魔力在哪里？”

嗯，神奇的事情要感谢`/usr/bin/qemu-aarch64-static`，它可以在容器映像本身中获得。这个二进制文件是 x86_64 二进制文件，不同于所有其他的 AArch64 二进制文件。Linux 内核会将 AArch64 二进制文件的执行转发给这个处理程序！

我们现在准备创建项目和 OpenShift 资源来处理容器映像的构建:

```
[alex@lenny ~]$ oc new-project test-arm-project
Now using project "test-arm-project" on server "https://192.168.42.213:8443".

[alex@lenny ~]$ oc new-app https://github.com/alezzandro/test-arm-container
--> Found Docker image 4734ae4 (3 days old) from Docker Hub for "multiarch/debian-debootstrap:arm64-stretch-slim"

    * An image stream tag will be created as "debian-debootstrap:arm64-stretch-slim" that will track the source image
    * A Docker build using source code from https://github.com/alezzandro/test-arm-container will be created
      * The resulting image will be pushed to image stream tag "test-arm-container:latest"
      * Every time "debian-debootstrap:arm64-stretch-slim" changes a new build will be triggered
    * This image will be deployed in deployment config "test-arm-container"
    * Port 8080/tcp will be load balanced by service "test-arm-container"
      * Other containers can access this service through the hostname "test-arm-container"
    * WARNING: Image "multiarch/debian-debootstrap:arm64-stretch-slim" runs as the 'root' user which may not be permitted by your cluster administrator

--> Creating resources ...
    imagestream.image.openshift.io "debian-debootstrap" created
    imagestream.image.openshift.io "test-arm-container" created
    buildconfig.build.openshift.io "test-arm-container" created
    deploymentconfig.apps.openshift.io "test-arm-container" created
    service "test-arm-container" created
--> Success
    Build scheduled, use 'oc logs -f bc/test-arm-container' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose svc/test-arm-container' 
    Run 'oc status' to view your app.
```

查看 OpenShift web 界面，我们可以看到容器映像已经成功构建，并且正在运行:

[![The running container image](img/4ca108ead9cf44d4bf9a46358aac9823.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/FireShot-Capture-046-OpenShift-Web-Console_-https___192.168.42.213_8443_consol.png)

我们甚至可以访问正在运行的容器，并尝试执行一些命令:

[![Executing some commands](img/65d6495294086215ed34c6ee4f157874.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/01/FireShot-Capture-047-OpenShift-Web-Console_-https___192.168.42.213_8443_consol.png)

正如您在前面的命令中所看到的，我们附加到了正在运行的容器，并验证了每个命令都是通过`/usr/bin/qemu-aarch64-static`代理的。

我们还检查了二进制文件是否是真正的 AArch64 架构。

我们现在可以在 Raspberry Pi 3 上尝试刚刚构建的容器图像。我选择 Fedora ARM Linux 作为基础操作系统。

首先，我从 [Fedora 的网站](https://arm.fedoraproject.org/)下载了图像后，用一个简单易用的工具安装了一张 SD 卡:

```
[alex@lenny Downloads]$ sudo arm-image-installer --addkey=/home/alex/.ssh/id_rsa.pub --image=/home/alex/Downloads/Fedora-Minimal-29-1.2.aarch64.raw.xz --relabel --resizefs --norootpass --target=rpi3 --media=/dev/sda
[sudo] password for alex: 
 ***********************************************************
 ** WARNING: You have requested the image be written to sda.
 ** /dev/sda is usually the root filesystem of the host. 
 ***********************************************************
 ** Do you wish to continue? (type 'yes' to continue)
 ***********************************************************
 = Continue? yes

=====================================================
= Selected Image:                                 
= /home/alex/Downloads/Fedora-Minimal-29-1.2.aarch64.raw.xz
= Selected Media : /dev/sda
= U-Boot Target : rpi3
= SELinux relabel will be completed on first boot.
= Root Password will be removed.
= Root partition will be resized
= SSH Public Key /home/alex/.ssh/id_rsa.pub will be added.
=====================================================
...
= Raspberry Pi 3 Uboot is already in place, no changes needed.
= Removing the root password.
= Adding SSH key to authorized keys.
= Touch /.autorelabel on rootfs.

= Installation Complete! Insert into the rpi3 and boot.
```

当我们全新的 Raspberry Pi 3 操作系统启动时，我们可以从正在运行的 Minishift 虚拟机中导出 Docker 映像。

我们将直接连接到虚拟机并运行一个`docker save`命令。我们可以使用这个技巧，因为我们在一个演示环境中；在一个真实的用例场景中，我们可能会导出内部的 OpenShift 注册表，以便让外部设备进行连接。

```
[alex@lenny ~]$ cdk-minishift ssh
Last login: Thu Jan 24 19:15:36 2019 from 192.168.42.1

[docker@minishift ~]$ docker save -o test-arm-container.tar 172.30.1.1:5000/test-arm-project/test-arm-container:latest

[alex@lenny Downloads]$ scp -i ~/.minishift/machines/minishift/id_rsa docker@`cdk-minishift ip`/home/docker/test-arm-container.tar
```

然后，我们可以将图像推送到 Raspberry Pi 并安装 Podman 来运行它！

不知道波德曼是什么？阅读更多关于 Red Hat Enterprise Linux 中的 [Podman。](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/)

```
[alex@lenny Downloads]$ scp test-arm-container.tar root@192.168.1.52:/root/test-arm-container.tar                              100% 214MB 450.1KB/s 08:07 

[alex@lenny Downloads]$ ssh root@192.168.1.52

[root@localhost ~]# cat /etc/fedora-release 
Fedora release 29 (Twenty Nine)

[root@localhost ~]# uname -a
Linux localhost.localdomain 4.19.15-300.fc29.aarch64 #1 SMP Mon Jan 14 16:22:13 UTC 2019 aarch64 aarch64 aarch64 GNU/Linux

[root@localhost ~]# dnf install -y podman

```

然后我们加载容器映像，最后运行它:

```
[root@localhost ~]# podman load -i test-arm-container.tar 
Getting image source signatures
Copying blob 36a049148cc6: 104.31 MiB / 104.31 MiB [=====================] 1m34s
Copying blob d56ce20a3f9c: 15.20 MiB / 15.20 MiB [=======================] 1m34s
Copying blob cf01d69beeaf: 94.53 MiB / 94.53 MiB [=======================] 1m34s
Copying blob 115c696bd46d: 3.00 KiB / 3.00 KiB [=========================] 1m34s
Copying config 478a2361357e: 5.46 KiB / 5.46 KiB [==========================] 0s
Writing manifest to image destination
Storing signatures
Loaded image(s): 172.30.1.1:5000/arm-project/test-arm-container:latest

[root@localhost ~]# podman images
REPOSITORY                                       TAG IMAGE ID CREATED SIZE
172.30.1.1:5000/arm-project/test-arm-container   latest 478a2361357e 4 hours ago 224 MB

[root@localhost ~]# podman run -d 172.30.1.1:5000/arm-project/test-arm-container:latest
cdbf0ac43a2dd01afd73220d5756060665df0b72a43bd66bf865d1c6149f325f

[root@localhost ~]# podman ps
CONTAINER ID  IMAGE                                         COMMAND CREATED STATUS PORTS  NAMES
cdbf0ac43a2d  172.30.1.1:5000/arm-project/test-arm-container:latest  /usr/sbin/apache2... 6 seconds ago Up 5 seconds ago       pedantic_agnesi

```

然后，我们可以测试 web 服务器:

```
[root@localhost ~]# podman inspect cdbf0ac43a2d | grep -i address
            "LinkLocalIPv6Address": "",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "GlobalIPv6Address": "",
            "IPAddress": "10.88.0.7",
            "MacAddress": "1a:c8:30:a4:be:2f"

[root@localhost ~]# curl 10.88.0.7:8080 2>/dev/null | head
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html >
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <title>Apache2 Debian Default Page: It works</title>
    <style type="text/css" media="screen">
  * {
    margin: 0px 0px 0px 0px;
    padding: 0px 0px 0px 0px;
```

最后，让我们检查内容，并将容器中的二进制文件与 Raspberry Pi 中的进行比较:

```
[root@localhost ~]# podman run -ti 172.30.1.1:5000/arm-project/test-arm-container:latest /bin/bash

root@8e39d1c28259:/# 
root@8e39d1c28259:/# id
uid=0(root) gid=0(root) groups=0(root)

root@8e39d1c28259:/# uname -a
Linux 8e39d1c28259 4.19.15-300.fc29.aarch64 #1 SMP Mon Jan 14 16:22:13 UTC 2019 aarch64 GNU/Linux

root@8e39d1c28259:/# cat /etc/debian_version 
9.6

root@8e39d1c28259:/# file /bin/bash

/bin/bash: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 3.7.0, BuildID[sha1]=29b2624b1e147904a979d91daebc60c27ac08dc6, stripped

root@8e39d1c28259:/# exit

[root@localhost ~]# file /bin/bash

/bin/bash: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 3.7.0, BuildID[sha1]=25dee020ab8f6525bd244fb3f9082a47e940b1e6, stripped, too many notes (256)
```

我们刚刚看到我们可以和波德曼一起运行这个容器。那么，为什么不应用我们在过去的文章中看到的关于 Podman 及其与`systemd`集成的一些技术呢？:)

阅读更多关于 Podman 和[管理集装箱化系统服务](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/)的信息。

## 额外资源

*   [通过 OpenShift 使用容器进行物联网边缘开发和部署:第 1 部分](https://developers.redhat.com/blog/2019/01/31/iot-edge-development-and-deployment-with-containers-through-openshift-part-1/)
*   [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)
*   [使用 Podman 管理集装箱化系统服务](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/)
*   没有守护进程的容器:RHEL 7.6 和 RHEL 8 测试版中的 Podman 和 Buildah
*   [pod man——下一代 Linux 容器工具](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)
*   [pod man 简介(Red Hat Enterprise Linux 7.6 中的新功能)](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/)
*   [定制 OpenShift Ansible 剧本包](https://developers.redhat.com/blog/2018/05/23/customizing-an-openshift-ansible-playbook-bundle/)

仅此而已。我希望你喜欢这个物联网解决方案！

## 关于亚历山德罗

![Alessandro Arrichiello](img/7c535812708e4fd93e70794bffaf765b.png)

Alessandro Arrichiello 是 Red Hat Inc .的解决方案架构师，他从 14 岁开始就对 GNU/Linux 系统充满热情，这种热情一直持续到今天。他使用过自动化企业 IT 的工具:配置管理和通过虚拟平台的持续集成。他现在致力于分布式云环境，涉及 PaaS (OpenShift)、IaaS (OpenStack)和流程管理(CloudForms)、容器构建、实例创建、HA 服务管理和工作流构建。

*Last updated: May 26, 2022*