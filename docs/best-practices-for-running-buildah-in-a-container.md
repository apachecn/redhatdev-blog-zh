# 在容器中运行 buildhr 的最佳实践

> 原文：<https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container>

将容器运行时分离到不同的工具中的一个很酷的事情是，您可以开始将它们组合起来，以帮助保护彼此。

很多人喜欢在像 [Kubernetes](https://kubernetes.io/) 这样的系统中构建 OCI/容器图像。假设你有一个持续构建容器映像的 CI/CD 系统，像[Red Hat open shift](https://developers.redhat.com/openshift/)/Kubernetes 这样的工具对于分配构建的负载会很有用。直到最近，大多数人都将 Docker 套接字泄漏到容器中，然后允许容器做`docker build`。正如我多年前指出的那样，这是你能做的最危险的事情之一。给予用户对系统或 sudo 的 root 访问权限而不需要密码，比允许访问 Docker 套接字更安全。

因此，许多人一直试图在容器中运行 Buildah。我们一直在关注和回答这方面的问题。我们已经构建了一个我们认为在容器内部运行 Buildah 的最佳方式的[示例](https://github.com/containers/Demos/tree/master/running/BuildahInPodman)，并在 quay.io/buildah 的[公开了这些容器图像。](https://quay.io/buildah)

## 设置

这些图像基于 Buildah repo `buildahimage `目录中提供的 docker 文件:[https://github . com/containers/Buildah/tree/master/contrib/buildahimage](https://github.com/containers/buildah/tree/master/contrib/buildahimage)。

我将检查[稳定 Dockerfile 文件](https://github.com/containers/buildah/blob/master/buildahimage/stable/Dockerfile)。

```
# stable/Dockerfile
#
# Build a Buildah container image from the latest
# stable version of Buildah on the Fedoras Updates System.
# https://bodhi.fedoraproject.org/updates/?search=buildah
# This image can be used to create a secured container
# that runs safely with privileges within the container.
#
FROM fedora:latest

# Don't include container-selinux and remove
# directories used by dnf that are just taking
# up space.
RUN yum -y install buildah fuse-overlayfs --exclude container-selinux; rm -rf /var/cache /var/log/dnf* /var/log/yum.*

# Adjust storage.conf to enable Fuse storage.
RUN sed -i -e 's|^#mount_program|mount_program|g' -e '/additionalimage.*/a "/var/lib/shared",' /etc/containers/storage.conf

```

我们在容器内部使用 [fuse-overlay](https://github.com/containers/fuse-overlayfs) 程序，而不是使用主机内核覆盖。原因是，目前内核覆盖挂载需要 SYS_ADMIN 功能，我们希望能够运行我们的 Buildah 容器，而不需要比普通根容器更多的特权。Fuse-overlay 工作得很好，比使用 VFS 存储驱动程序给我们带来了更好的性能。注意，使用 Fuse 需要运行 Buildah 容器的人提供/dev/fuse 设备。

```
podman run --device /dev/fuse quay.io/buildahctr ...
RUN mkdir -p /var/lib/shared/overlay-images /var/lib/shared/overlay-layers; touch /var/lib/shared/overlay-images/images.lock; touch /var/lib/shared/overlay-layers/layers.lock

```

在这里，我为其他商店建立了一个目录。[容器/存储器](https://github.com/containers/storage)支持添加额外只读图像存储器的概念。例如，您可以在一台机器上设置一个覆盖存储区域，然后 NFS 将该存储装载到另一台机器上，这样就可以使用映像，而不必将其删除。我们计划使用这个存储，这样我们就可以从主机上卷装一些要在容器中使用的映像存储。

```
# Set up environment variables to note that this is
# not starting with user namespace and default to
# isolate the filesystem with chroot.
ENV _BUILDAH_STARTED_IN_USERNS="" BUILDAH_ISOLATION=chroot

```

最后，我们默认 Buildah 容器运行 chroot 隔离。设置环境变量`BUILDAH_ISOLATION`告诉 Buildah 默认使用 chroot。我们不需要额外的隔离运行，因为我们已经在一个容器中运行了。让 Buildah 创建自己的名称空间分隔的容器需要 SYS_ADMIN 特权，并且需要我们放松运行容器上的 SELinux 和 SECCOMP 规则，这违背了在锁定的容器中运行构建的目的。

## 在容器中运行 buildhr

我们设计上面的 Buildah 容器映像的方式允许我们在如何启动容器上获得最大的灵活性。

### 安全性与速度

在计算机安全的世界里，一个进程运行的速度和我们能够包装它的安全性之间总是有一场战斗。在构建容器时，我们有同样的权衡。在下一节中，我将描述速度和安全性之间的权衡。

上面的容器映像将保持其在`/var/lib/containers`中的存储，因此我们需要将内容卷安装到这个目录中，这个卷可以显著地改变构建容器映像的速度。

让我们看三个潜在的例子。

1.为了最安全，我可以为每个容器创建一个新的容器/映像目录，并将其卷挂载到容器中。我们还将把上下文目录放入`/build`下的容器中:

```
# mkdir /var/lib/containers1
# podman run -v ./build:/build:z -v /var/lib/containers1:/var/lib/containers:Z quay.io/buildah/stable\
buildah  -t image1 bud /build
# podman run -v /var/lib/containers1:/var/lib/containers:Z quay.io/buildah/stable buildah  push \ image1 registry.company.com/myuser
# rm -rf /var/lib/containers1

```

**安全性:**在这个容器中运行的 Buildah 被完全锁定，它使用 dropped 功能、SECOMP enforcing 和 SELinux enforcing 运行。您甚至可以通过添加类似于`--uidmap 0:100000:10000`的东西来运行这个带有用户名称空间分离的容器。

**性能:**这是性能最低的，因为它需要从容器注册表中下载所有要使用的图像，而且它不能利用缓存。当 Buildah 容器完成时，它应该将图像推送到注册表并销毁内容。您可能希望基于此新容器映像构建的未来容器映像将必须从注册表中提取新映像，因为该映像已从主机中删除。

2.如果我们想达到 Docker 的性能，我们可以将主机容器/存储卷装载到容器中。

```
# podman run -v ./build:/build:z -v /var/lib/containers:/var/lib/containers --security-opt label:disabled quay.io/buildah/stable buildah  -t image2 bud /build
# podman run -v /var/lib/containers:/var/lib/containers --security-opt label:disabled \ quay.io/buildah/stable buildah push image2 registry.company.com/myuser

```

**安全性:**这是构建容器最不安全的方式，容器被允许修改主机上的容器存储，并可能导致 Podman 或 CRI-O 对流氓映像做一些事情。此外，为了实现这一点，我必须禁用 SELinux 分离。SELinux 会阻止 Buildah 容器进程与主机存储交互。请注意，这仍然比使用 Docker 套接字运行要好，因为容器仍然被其他安全特性锁定，无法在主机上轻松启动容器。

**性能:**这是最好的性能，因为它可以利用缓存。如果 Podman 或 CRI-O 以前将映像拉至主机，容器内部的 Buildah 进程将不需要重新拉映像。基于此映像的未来构建也可以利用缓存。

3.构建容器的第三种方法是创建一个项目容器映像目录，并在项目中的所有容器之间共享该映像目录。

```
# mkdir /var/lib/project3
# podman run --security-opt label:level=s0:C100, C200 -v ./build:/build:z \
-v /var/lib/project3:/var/lib/containers:Z quay.io/buildah/stable buildah  -t image3 bud /build
# podman run --security-opt label:level=s0:C100, C200 \
-v /var/lib/project3:/var/lib/containers quay.io/buildah/stable buildah push image3 \ registry.company.com/myuser 
```

在第三个例子中，我没有在运行之间删除项目目录(`/var/lib/project3`)，所以同一个项目中的未来构建可以利用缓存。

**安全性:**这是安全性的中间地带。容器不能访问主机的内容，也不能让 Podman/CRI-O 通过将内容写入它们的映像存储来做坏事。容器会影响同一项目中的其他容器构建器。

**性能:**这种设置的性能可能不如与主机共享缓存，因为它不能利用以前由 Podman/CRI-O 提取的映像。但是一旦一个 Buildah 提取映像，所有其他构建都可以利用该映像。

### 其他商店

[Containers/storage](https://github.com/containers/storage) 有一个很酷的特性叫做 additional stores，它允许容器引擎在运行和构建容器时使用外部容器覆盖映像存储(只读)。基本上，您可以将一个或多个只读存储添加到 storage.conf 文件中，然后在运行容器时，容器引擎将在每个存储中搜索您想要运行的映像。而且，如果没有商店找到图像，它只会从注册表中提取图像。容器引擎将只能写入其单个可写存储。

如果您回过头来看看我们用来构建 quay.io/buildah/stable 映像的 docker 文件，您会看到这几行:

```
# Adjust storage.conf to enable Fuse storage.
RUN sed -i -e 's|^#mount_program|mount_program|g' -e '/additionalimage.*/a "/var/lib/shared",' /etc/containers/storage.conf
RUN mkdir -p /var/lib/shared/overlay-images /var/lib/shared/overlay-layers; touch /var/lib/shared/overlay-images/images.lock; touch /var/lib/shared/overlay-layers/layers.lock

```

第一行修改容器图像内部的`/etc/containers/storage.conf`。它告诉存储驱动程序使用`/var/lib/shared`目录中的“additionalimagestores”。在下一行中，我创建了共享目录，并添加了几个锁文件来保持容器/存储的良好状态。基本上，这是创建一个空的容器图像存储。

如果我们在这个目录之上的容器/存储器中进行卷挂载，那么 Buildah 将能够使用这些映像。

如果我们回到上面的例子，我们能够利用 Buildah 映像中的 hosts 容器/store，我们可以获得最佳性能，因为 Podman/CRI-O 以前可能已经拉下了映像。但是我们得到了最差的安全性，因为容器可以写给商店。有了额外的图像，我们可以两全其美。

```
# mkdir /var/lib/containers4
# podman run -v ./build:/build:z -v /var/lib/containers/storage:/var/lib/shared:ro -v \ /var/lib/containers4:/var/lib/containers:Z  quay.io/buildah/stable \
 buildah  -t image4 bud /build
# podman run -v /var/lib/containers/storage:/var/lib/shared:ro  \
-v >/var/lib/containers4:/var/lib/containers:Z quay.io/buildah/stable buildah push image4 \ registry.company.com/myuser
# rm -rf /var/lib/continers4

```

注意我是如何将主机上的`/var/lib/containers/storage`挂载到只读容器中的`/var/lib/shared`上的。当 Buildah 在容器中运行时，它可以利用任何以前由 Podman/CRI-O 提取的图像来加快速度，但它仍然只能写入自己的存储。还要注意，我现在可以在启用 SELinux 容器分离的情况下做到这一点。

#### 非常酷

这样做的一个潜在问题是，您不应该从底层存储中删除任何映像。如果这样做，可能会导致 Buildah 容器爆炸。

#### 但这还不是全部...

附加商店甚至比那更好。您可以设置一个存储所有容器映像的网络共享存储。然后，您可以将这个存储共享给所有的 Buildah 容器。假设您有一百个映像，您的 CI/CD 系统经常使用它们来构建容器映像。您可以在一台主机上预装所有映像的存储。然后，你可以使用你最喜欢的网络存储工具(NFS，Gluster，Ceph，ISCSI，S3...)并与所有 Buildah 或 Kubernetes 节点共享存储。

只需在`/var/lib/shared`将这个网络存储卷装载到 Buildah 容器中，您的 Buildah 容器就不再需要提取映像了。它们都是预先填充的，您可以开始工作了。

当然，您的 Kubernetes 和容器基础设施也可以利用这一点，在任何地方启动和运行容器，而根本不需要提取图像。我甚至可以想象一个容器注册中心，当接收到推送给它的容器映像时，它会将容器映像展开到共享存储上。然后，所有节点都可以立即访问更新后的映像。

我听说过巨大的数千兆字节的容器图像。使用额外的存储意味着您不再需要在您的环境中复制它们，并且您的容器启动时间将是瞬时的。

在我的下一篇文章中，我将介绍我们正在开发的一个新功能——覆盖卷挂载——它将使构建映像的速度更快。

## 结论

在 Kubernetes/CRI-O 或 Podman，甚至 Docker 的容器中运行 Buildah 很容易，而且比在 docker.socket 中泄漏要安全得多。

额外的存储可以用来帮助加速甚至消除下拉容器图像的需要。

我们还制作了一个演示来帮助说明这里讨论的概念:

*Last updated: October 17, 2019*