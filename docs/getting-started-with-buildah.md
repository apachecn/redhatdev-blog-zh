# buildhr 入门

> 原文：<https://developers.redhat.com/blog/2021/01/11/getting-started-with-buildah>

如果您希望在没有安装完整的容器运行时或守护进程的情况下构建开放容器倡议(OCI)容器映像，那么 [Buildah](https://buildah.io/) 是完美的解决方案。现在，Buildah 是一个开源的、基于 Linux 的工具，可以构建 Docker-和 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 兼容的映像，并且很容易集成到脚本和构建管道中。此外，Buildah 还具有与[波德曼](https://podman.io/)、[斯科佩奥](https://github.com/containers/skopeo)和 [CRI-O](https://cri-o.io/) 的重叠功能。

Buildah 能够从头开始创建一个工作的[容器](https://developers.redhat.com/topics/containers/)，也可以从一个预先存在的 Dockerfile 文件中创建。另外，由于它不需要守护进程，所以在构建容器映像时，您永远不必担心 Docker 守护进程的问题。

让我们通过一些真实的例子来展示开始使用 Buildah 是多么容易，以及创建一个容器映像是多么容易。

## 安装 buildhr

如果你运行的是[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/overview)(RHEL)8，请遵循以下步骤。对于 Fedora 用户，一定要将`yum`替换为`dnf`:

```
$ yum -y install buildah
```

但是，如果你没有 Linux 可用，你可以用 Katacoda 在线使用 [Buildah。](https://www.katacoda.com/courses/containers-without-docker/building-container-images-with-buildah)

## 基本命令

为了了解 Buildah，我们先来研究一些基本的命令。命令`buildah --version`将输出我们的 Buildah 安装的当前版本，如果您遇到困难，`buildah --help`将会帮助您。

例如，为了从存储库中提取容器图像，使用`from`变量。例如，如果您最喜欢的 Linux 发行版是 CentOS:

```
$ buildah from centos
```

在提取图像并将其存储在主机上之后，通过运行`buildah images`列出我们当前的图像。这种行为类似于 Podman 和 Docker，因为许多命令是交叉兼容的。要获得我们正在运行的容器的列表，一旦图像拉取完成就提供这些容器，使用`buildah containers`。例如，参见图 1。

[![The output of the command &quot;buildah containers&quot;, showing CONTAINER ID, BUILDER, IMAGE ID, IMAGE NAME, and CONTAINER #](img/ed2d56cdd8b9d952ba81b2372cb676aa.png "Buildah containers")](/sites/default/files/blog/2020/08/Buildah-containers.png)

最后，既然我们已经拉出并显示了一个容器，那么让我们用`buildah rm -all`清理并移除正在运行的容器。但是，一定要小心，因为 Buildah 能够删除正在运行的容器，而 Docker 不能。

## 构建容器

是时候动手使用 Buildah 并构建一个将在容器中运行的 Apache web 服务器了。首先，让我们拉一个 CentOS 基础映像并开始工作:

```
$ buildah from centos
```

您将在控制台中看到默认的图像名称作为输出，如`centos-working-container`，这使我们能够在指定的容器中运行命令。对于我们的例子，我们将安装一个 [httpd](https://httpd.apache.org/docs/current/programs/httpd.html) 包，可以使用下面的命令来完成:

```
$ buildah run centos-working-container yum install httpd -y
```

一旦我们安装了`httpd`，我们就可以把注意力放在创建一个指向我们 web 服务器的主页上，通常称为`index.html`文件。要创建一个简单的文件而不必担心格式，使用下面的`echo`命令:

```
$ echo "Hello from Red Hat" > index.html
```

另外，在创建这个新文件之后，让我们用 Buildah `copy`函数将它复制到我们当前的工作容器中。还包括可公开访问的文件的默认位置:

```
$ buildah copy centos-working-container index.html /var/www/html/index.html
```

要启动这个容器，我们必须为一个容器配置一个入口点，这个入口点用于在容器开始时启动`httpd`,并将其保持在前台:

```
$ buildah config --entrypoint "/usr/sbin/httpd -DFOREGROUND" centos-working-container
```

最后，让我们`commit`我们对容器的更改，并准备将它推送到您想要的任何容器注册表(例如 [Docker and Quay.io](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/) ):

```
$ buildah commit centos-working-container redhat-website
```

你的`redhat-website`图像已经准备好[与波德曼](https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container/)一起运行，或者被推送到你选择的注册表。

## 使用 Dockerfile 文件构建

Buildah 的另一个重要部分是使用 docker 文件构建图像的能力，而`build-using-dockerfile`或`bud`命令可以做到这一点。让我们以一个示例 Dockerfile 作为输入，并输出一个 OCI 图像:

```
# CoreOS Base
FROM fedora:latest
# Install httpd
RUN echo "Installing httpd"; yum -y install httpd
# Expose the default httpd port 80
EXPOSE 80
# Run httpd
CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

一旦我们将这个文件保存为本地目录中的`Dockerfile`，我们就可以使用`bud`命令来构建映像:

```
$ buildah bud -t fedora-httpd
```

为了仔细检查我们的进度，让我们运行`buildah images`并确保我们可以看到新的`fedora-httpd`映像驻留在我们的本地主机存储库中。现在，你可以自由地用 Podman 再次[运行图像，或者把它推送到你最喜欢的注册表。](https://developers.redhat.com/blog/2019/08/14/best-practices-for-running-buildah-in-a-container/)

## 结论

干得好！我们已经从头开始构建了一个容器，也从一个预定义的 docker 文件开始。Buildah 是一种轻量级的、灵活的创建容器映像的方法，不需要安装运行时或守护进程。

您可以通过设置这个 Katacoda 场景来继续使用 Buildah 进行实验，它会在您的浏览器中为您提供一个交互式环境。

如果需要容器编排，可以使用 Buildah 搭配 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 或者 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 。要开始使用这些平台，请参见 kubernetesbyexample.com 和 learn.openshift.com[的](https://learn.openshift.com/)[。](https://kubernetesbyexample.com/)

要观看您在这里看到的许多示例的交互式演示，请观看此视频:

No video provider was found to handle the given URL. See [the documentation](https://www.drupal.org/node/2842927) for more information.

## 资源

了解关于 Buildah 的更多信息:

*   面向 Docker 用户的 Podman 和 Buildah】(威廉·亨利)
*   在容器中运行 Buildah 的最佳实践
*   在 Podman 容器中构建并运行 Buildah

*Last updated: May 19, 2022*