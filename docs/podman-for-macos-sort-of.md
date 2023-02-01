# 苹果电脑的博德曼

> 原文：<https://developers.redhat.com/blog/2020/02/12/podman-for-macos-sort-of>

我有一个问题。我的日常笔记本电脑是 MacBook Pro，这很棒，除非你想双启动到 Linux 并在容器上开发。虽然安装[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview)很简单，但我真正需要的是一种在 macOS 上运行 Buildah、Podman 和 skopeo 的方法，而不必给 Linux VM 浇水和喂食。

不用再看了:机器人已经在某种程度上解决了这个问题。

## 机器人-机器

Podman-machine 启动一个虚拟机，该虚拟机已经简化了 Podman、Buildah 和 skopeo 包。开发人员发布了两种 VM 风格:内存微内核和 Fedora 版本。

你可以选择为 xhyve 之类的管理程序编译额外的驱动程序支持，但我会推荐 VirtualBox，因为它似乎工作得更流畅。

## 入门指南

我的指示是基于这里的官方指示。本指南还假设您已经安装了 VirtualBox。

从下载最新的`podman-machine`二进制文件开始。在撰写本文时，最新版本是 v0.16:

```
$ curl -L https://github.com/boot2podman/machine/releases/download/v0.16/podman-machine.darwin-amd64 --output /usr/local/bin/podman-machine
chmod +x 

```

### 设置您的虚拟机

然后，创建一个`boot2podman`虚拟机。我使用的是一个 Fedora 31 虚拟机，内存为 4GB，我将本地的`~/Code`目录连接到这个虚拟机。

我将图像更新到 Fedora 31，并允许无根图像构建。这张照片应该会被官方回购。同时，我参考了下面的开发版本:

```
$ podman-machine create --virtualbox-boot2podman-url https://github.com/snowjet/boot2podman-fedora-iso/releases/download/d1bb19f/boot2podman-fedora.iso --virtualbox-memory="4096" --virtualbox-share-folder ~/Code:code fedbox

```

现在，您有了一个带有用于容器映像的持久磁盘的 VM，但是它在内存中运行操作系统。您可以在`/sf_code`登录虚拟机并查看您的共享目录:

```
$ podman-machine ssh fedbox

ls /sf_code
total 12
drwxrwx---.  1 root vboxsf  128 Jan 13 21:15 .
dr-xr-xr-x. 18 root root   4096 Jan 14 22:42 ..
drwxrwx---.  1 root vboxsf  480 Aug 28 05:40 container-proj

```

### 设置您的容器

现在，让我们运行一个容器并与之通信:

```
$ podman-machine ssh fedbox
$ podman run -p 8080:80/tcp --rm httpd
Trying to pull docker.io/library/httpd...
Getting image source signatures
Copying blob 27298e4c749a done
Copying blob 354e6904d655 done
Copying blob 36412f6b2f6e done
Copying blob 10e27104ba69 done
Copying blob 8ec398bc0356 [======================================] 25.8MiB / 25.8MiB
Copying config c2aa7e16ed [======================================] 7.2KiB / 7.2KiB
Writing manifest to image destination
Storing signatures
...
[Thu Jan 16 01:28:19.051375 2020] [core:notice] [pid 1:tid 140000832345216] AH00094: Command line: 'httpd -D FOREGROUND'

```

在另一个终端中，运行:

```
$ podman-machine ip fedbox
192.168.99.122
$ curl http://192.168.99.122:8080
It works!

```

最后，您可以在 Mac 上创建容器并与它们通信。

### 关闭您的工作区

要停止并清理您的工作区，请运行:

```
$ podman-machine stop fedbox
$ podman-machine rm fedbox

```

现在，您可以在 Mac 上轻松构建、运行和推送容器。

*Last updated: June 29, 2020*