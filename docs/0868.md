# 在 podman 容器中构建并运行 buildhr

> 原文：<https://developers.redhat.com/blog/2019/04/04/build-and-run-buildah-inside-a-podman-container>

去年圣诞节，我送给妻子一套类似俄罗斯套娃的套娃。如果你不熟悉它们，它们由一个木制娃娃组成，打开后会出现另一个娃娃，在里面你会发现另一个娃娃，以此类推，直到你看到所有娃娃中最小的、通常是最华丽的娃娃。这个概念让我想到了嵌套容器。

我想我会尝试使用 [Podman](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/) 构建我自己的嵌套容器来创建一个容器，在这个容器中我可以进行 [Buildah](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/) 开发，还可以旋转 Buildah 容器和图像。一旦这个 Podman 容器被创建，我就可以把它转移到任何支持 Podman 的 [Linux](https://developers.redhat.com/topics/linux/) 平台上，并在其上进行 Buildah 开发。在本文中，我将展示如何设置它。

## 准备环境

我在一个新安装的 Fedora 29 虚拟机上开始了这个实验，用`dnf -y install podman container-selinux --enablerepo updates-testing`在上面安装了最新的 Podman 和 container-selinux。这给了我 pod man 1 . 1 . 2 版和 container-selinux 2.85-1 版。

因为容器和容器中的容器都将使用 fuse-overlayfs，所以它们不会乐于尝试在彼此之上安装各自的目录。因此，第一步是在要使用的容器中创建一个目录，我将其命名为`/var/lib/mycontainer`:

```
# mkdir /var/lib/mycontainer
```

## 波德曼容器创建

然后我创建了下面的 Dockerfile，它将提取 Fedora，设置 GOPATH，安装 Buildah 依赖项，使用 git 克隆`/root/buildah`目录中的项目，最后使用 sed 更新`/etc/container/storage.conf`以取消对 mount_program 的注释:

```
# FILE=~/Dockerfile.cinc
# /bin/cat <<EOM >$FILE
FROM fedora:latest
ENV GOPATH=/root/buildah

RUN dnf -y install \
make \
golang \
bats \
btrfs-progs-devel \
device-mapper-devel \
glib2-devel \
gpgme-devel \
libassuan-devel \
libseccomp-devel \
ostree-devel \
git \
bzip2 \
go-md2man \
runc \
fuse-overlayfs \
fuse3 \
containers-common; \
mkdir /root/buildah; \
git clone https://github.com/containers/buildah /root/buildah/src/github.com/containers/buildah

RUN sed -i -e 's|#mount_program = "/usr/bin/fuse-overlayfs"|mount_program = "/usr/bin/fuse-overlayfs"|' /etc/containers/storage.conf
EOM

```

接下来，我们使用 Dockerfile 文件创建图像。(注意行尾的难见句号。)该命令可能需要 5 到 10 分钟才能完成，并且在接近结束时似乎会挂起一会儿，所以请耐心等待。这是一个伟大的命令，当你去喝茶的时候，你可以开始执行它。

```
# podman build -t buildahimage -f ~/Dockerfile.cinc .
```

举重就这样了。让我们创建一个 Podman 容器，我们将在其中进行 Buildah 开发。下面的命令创建一个名为`buildahctr`的容器，将主机的`mycontainer`挂载到容器的`containers`目录，使用主机的网络运行分离的容器，关闭容器中的 label 和 seccomp 限制，最后做一点 shell hackery 来保持容器启动和运行。

```
# podman run --detach --name=buildahctr --net=host --security-opt label=disable --security-opt seccomp=unconfined --device /dev/fuse:rw -v /var/lib/mycontainer:/var/lib/containers:Z   buildahimage sh -c 'while true ;do wait; done'

```

## buildhr 开发

很好，现在我们有了一个运行 Fedora 的容器。让我们在上面编译并安装 Buildah。下面的命令将把我们带到容器内部的命令行。

```
# podman exec -it buildahctr /bin/sh

```

既然我们已经进入了容器，是时候运行一些标准的 make、git 和 Buildah 了。(注意，接下来的五个命令是在`sh-4.4#`提示符下运行的；为了便于剪切和粘贴，我删除了几个提示。)

```
sh-4.4# cd /root/buildah
export GOPATH=`pwd`
cd /root/buildah/src/github.com/containers/buildah
make
make install

sh-4.4# buildah from alpine
alpine-working-container

sh-4.4# buildah images
REPOSITORY               TAG    IMAGE  ID    CREATED    SIZE
docker.io/library/alpine latest 5cb3aa00f899 9 days ago 5.79 MB

```

所以现在我们已经从一个 Podman 容器中编译、安装并运行了 Buildah。

接下来，让我们对 Buildah 源代码做一个快速的更改，看看我们是否可以在适当的地方运行它。用 vi 或者自己喜欢的编辑器来改`cmd/buildah/images.go`。搜索`outputHeader()`函数(219 行附近)，找到其中的一行:`format := "table {{.Name}}\t{{.Tag}}\t"`。将单词“table”从该行中删除，使其成为`format := "{{.Name}}\t{{.Tag}}\t”`。保存文件，退出，然后再按`make`和`make install`键。

```
sh-4.4# vi cmd/buildah/images.go
sh-4.4# make
sh-4.4# make install

```

现在，如果您运行`buildah images`，您应该会看到我们已经让所有的输出都像`--quiet`选项一样运行，这并不显示表头。

```
sh-4.4# buildah images
docker.io/library/alpine  latest 5cb3aa00f899 9 days ago   5.79 MB

```

## 在 podman 容器中运行 buildhr 容器

最后一个有趣的地方是，让我们看看是否可以使用修改后的 Buildah 代码在这个 Podman 容器中运行一个 Buildah 容器。我们将做一些简单的事情，列出`/`目录的内容。

```
sh-4.4# buildah from --name myalpine alpine
myalpine

sh-4.4# buildah run --isolation=chroot myalpine ls /
bin   dev   etc  home   lib   media   mnt  opt   proc   root run   sbin   srv   sys   tmp   usr   var

```

## 可移植的 buildhr 开发环境

如果您已经做到了这一步，那么您已经使用 Podman 构建了一个能够进行 Buildah 开发的容器。该容器还可以构建容器，然后运行这些容器。我在这个例子中使用了 Buildah，但是我也可以使用 Podman 来构建内部容器。不管选择的内部工具是什么，我现在有了一个可以在内部构建和运行容器的容器，就像 Matryoshka 玩偶一样。更好的办法是，我可以提交这个容器，并把它推到 Quay.io 或另一个容器注册中心，然后把它拉下来，在另一台 Fedora 机器甚至另一个 Linux 平台上运行。我制作了一个完全可移植的 Buildah 开发环境。

我希望这个练习能让你思考如何使用 Podman 和 Buildah 在你自己的商店中创建一个更加灵活和动态的环境。另一个博客正在开发中，它将展示如何在不作为根用户的情况下做到这一点，敬请关注！

附注:在这次演示中，没有任何守护进程受到损害或被部署。

*Last updated: October 17, 2019*