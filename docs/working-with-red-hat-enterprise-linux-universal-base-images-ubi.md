# 使用 Red Hat Enterprise Linux 通用基础映像(UBI)

> 原文：<https://developers.redhat.com/blog/2019/05/31/working-with-red-hat-enterprise-linux-universal-base-images-ubi>

如果你像我一样——一名开发人员，与依赖可靠的[Red Hat Enterprise Linux(RHEL)](https://developers.redhat.com/rhel8/)的客户一起工作，使用容器化的应用程序，也喜欢使用 [Fedora Linux](https://getfedora.org/en/) 作为他们的桌面操作系统——你会对[通用基础映像(UBI)](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image) 的宣布感到兴奋。本文通过为一个简单的 PHP 应用程序构建[容器](https://developers.redhat.com/blog/category/containers/)映像，展示了 UBI 的实际工作方式。

使用 UBI，您可以构建和重新分发基于 Red Hat Enterprise Linux 的容器映像，而无需订阅 Red Hat。基于 UBI 的容器映像的用户不需要订阅 Red Hat。为您的社区项目或喜欢自助服务的客户创建基于 CentOS 的容器映像不再需要额外的工作。

我在我个人的 Fedora 29 系统上测试了所有这些步骤，它们应该可以在任何 Linux 发行版上运行。我也是新容器工具的忠实粉丝，比如 [Podman、](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)，你最喜欢的 Linux 发行版应该会有这些工具。如果您在 Windows 或 MacOS 系统上工作，您可以用 Docker 替换 Podman 命令。

## 使用 Red Hat 的公共注册表

容器目录中的下载说明假设使用 Red Hat 的基于术语的注册中心，这是一个私有注册中心。这个帖子上的说明也做了同样的假设。如果您的目的只是构建可自由再发行的基于 UBI 的图像，您可以选择使用 Red Hat 的公共注册表，而不是 Red Hat 的私有注册表。为此:

*   跳过在 Red Hat Developers 网站注册的步骤，创建一个服务帐户，并登录到 Red Hat Container Registry。
*   在所有的 Podman 命令中，将私有注册表主机名 *registry.redhat.io* 替换为公共注册表主机名【registry.access.redhat.com】T2。
*   在 docker 文件的 *FROM* 指令中使用公共注册表主机名*registry.access.redhat.com*。

## 下载 UBI 容器映像

首先访问[红帽容器目录](https://access.redhat.com/containers)并搜索 UBI。在第一批结果中，您会发现“[ubi 7/ubi-Red Hat Universal Base Image 7 by Red Hat，Inc.](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/ubi7/ubi) ”，我将使用它作为我的基本映像，因为我还没有升级到新的 RHEL8 版本。但是，请注意，已经有基于 RHEL8 的 UBI 图像可供您使用。有了这些，您可以试验新的 RHEL8 版本，而无需在虚拟机中安装完整的操作系统。

ubi7/ubi 的容器目录页声明:“这个基本映像可以自由再分发，但是 Red Hat 只通过订阅 Red Hat 产品来支持 Red Hat 技术。”并不是所有的 UBI 容器映像都可以自由地重新发布，所以要小心。使用 UBI，您或您的用户可以随时决定成为 Red Hat 客户，这使您有权获得 Red Hat 的支持服务，而无需重建您的容器映像或重新部署您的应用程序以成为基于 RHEL 的。这是使用基于 CentOS 的容器映像无法轻易做到的。

如果你点击*获取这个图片*，你会看到你需要认证才能从 Red Hat 下载 UBI 图片。如果你还没有在 [Red Hat Developers 网站注册，](https://developers.redhat.com)请注意，它是免费的，你可以免费使用许多 Red Hat 技术。

注册并登录红帽开发者后，访问[红帽客户门户](https://access.redhat.com/terms-based-registry/)。在那里，您生成一个允许您下载 Red Hat 容器图像的服务帐户。接下来，剪切并粘贴服务帐户名及其自动生成的令牌，以登录到 Red Hat 容器注册表:

```
$ sudo podman login -u "your service account name" -p "your very long token" registry.redhat.io
```

现在你可以启动你的第一个基于 UBI 的容器了:

```
$ sudo podman run -it registry.redhat.io/ubi7/ubi bash
...
[root@8285feb36cdb /]#
```

## 探索 UBI 容器映像

前面的命令启动了一个容器中的 Bash shell，该容器提供了一个基本的 Red Hat Enterprise Linux 操作系统。您的应用程序容器映像可能需要您添加一些包，例如，一个编程语言运行时。在 UBI 之前，你需要订阅红帽来下载 RHEL 软件包。现在 UBI 自带了一套可自由再发行内容的包存储库。这些库提供了 RHEL 包的子集。让我们来看看这些包存储库:

```
[root@8285feb36cdb /]# yum repolist
Loaded plugins: ovl, product-id, search-disabled-repos, subscription-manager
This system is not registered with an entitlement server. You can use subscription-manager to register.
...
repo id                                   repo name                                                                                         status
ubi-7/x86_64                              Red Hat Universal Base Image 7 Server (RPMs)                                                      832
ubi-7-optional/x86_64                     Red Hat Universal Base Image 7 Server - Optional (RPMs)                                            18
ubi-7-rhah/x86_64                         Red Hat Universal Base Image Atomic Host (RPMs)                                                     3
ubi-7-rhscl/x86_64                        Red Hat Software Collections for Red Hat Universal Base Image 7 Server (RPMs)                     320
repolist: 1173
```

请注意，RHEL7 的 [Red Hat 软件集合](https://developers.redhat.com/products/softwarecollections/overview)的一个子集在 UBI 仓库中是可用的。这为您提供了 PHP、Node.js 和其他编程语言运行时的最新支持版本。

如果您是 Java 开发人员，请注意 UBI 同时提供了新版本和旧版本的 OpenJDK:

```
[root@8285feb36cdb /]# yum search openjdk
```

当主机系统注册了有效的 Red Hat 订阅时，UBI 映像提供对整个 RHEL 的访问。只有当您的主机系统未注册时，您才能使用 UBI 库:

```
[root@8285feb36cdb /]# ls /etc/yum.repos.d/
redhat.repo  ubi.repo
```

如果你对 UBI 库感兴趣，可以查看一下配置文件:

```
[root@8285feb36cdb /]# vi /etc/yum.repos.d/ubi.repo
```

我们已经完成了对基本 UBI 容器图像的探索，现在可以退出文本编辑器和图像外壳了。

## 探索 UBI 最小容器映像

请注意， *ubi* 容器映像包含一些您在应用程序容器映像上可能不需要的工具，比如 vim 文本编辑器。如果你认为你不需要那些额外的命令，就切换到 *ubi7/ubi-minimal* 容器映像。然而，拥有这些额外的工具在容器映像的开发过程中可能会有所帮助。

启动一个运行 *ubi7/ubi-minimal* 容器映像的新容器。您会注意到它显示了不同的默认 Bash 提示符:

```
$ sudo podman run -it registry.redhat.io/ubi7/ubi-minimal bash
...
bash-4.2#
```

这不是那种基于 busybox 的最小容器映像。这是一个真正的 RHEL 集装箱形象，由真正的 RHEL 包装。对于 UBI 和 subscription 客户，这些软件包以相同的方式进行强化和更新。

*ubi* 和 *ubi-minimal* 映像之间的主要区别在于，前者提供了完整的 yum 工具集。Yum 增加了一些依赖项，比如 Python 包。相比之下，第二个提供了 microdnf 作为替代。microdnf 工具在与 Yum 相同的存储库中工作，但是只提供安装、更新和删除软件包的能力:

```
bash-4.2# ls /etc/yum.repos.d/
redhat.repo  ubi.repo
bash-4.2# microdnf --help
Usage:
 microdnf [OPTION?] COMMAND

Commands:
 clean - Remove cached data
 install - Install packages
 remove - Remove packages
 update - Update packages
...
```

我们已经完成了对最小 UBI 容器映像的探索。你现在可以退出你的图像外壳。现在让我们看看使用 UBI 的真实应用程序映像是什么样子的。

## 使用 UBI 的示例应用程序

我的测试应用程序可以从 [GitHub](https://github.com/flozanorht/testphp-ubi) 获得，如果你不想自己创建它的文件和输入它的内容。它只包含两个文件:一个 *Dockerfile* 和一个*index.php*脚本。

首先，您可以创建一个工作文件夹，并在其中创建一个受启发的“Hello，world”PHP 增强的网页，例如:

```
<html>
<body>
<?php print "Hello, world!\n" ?>
</body>
</html>
```

在同一个工作文件夹中，创建一个 docker 文件。Dockerfile 使用 UBI 包存储库从软件集合库中安装 Apache HTTPd 和 PHP:

```
FROM registry.redhat.io/ubi7/ubi

RUN yum -y install --disableplugin=subscription-manager \
  httpd24 rh-php72 rh-php72-php \
  && yum --disableplugin=subscription-manager clean all

ADD index.php /opt/rh/httpd24/root/var/www/html

RUN sed -i 's/Listen 80/Listen 8080/' \
  /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf \
  && chgrp -R 0 /var/log/httpd24 /opt/rh/httpd24/root/var/run/httpd \
  && chmod -R g=u /var/log/httpd24 /opt/rh/httpd24/root/var/run/httpd

EXPOSE 8080

USER 1001

CMD scl enable httpd24 rh-php72 -- httpd -D FOREGROUND
```

从没有活动订阅的机器(比如我的个人 Fedora 机器)构建时，*-disable plugin = subscription-manager*选项可以避免警告消息。

注意，我创建了一个 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 友好的容器图像。该映像在没有 root 权限和随机用户 id 的情况下工作。这是每个人都应该做的事情，不管他们的目标容器运行时是什么。与桌面操作系统的容器引擎和一些 Kubernetes 发行版不同，Red Hat OpenShift 默认情况下拒绝运行需要提升权限的容器。

进入您的工作文件夹，使用 Podman 构建容器映像:

```
$ sudo podman build -t php-ubi .
```

这很酷，不是吗？从一个非 RHEL 系统构建一个基于 RHEL 的容器映像，不需要订阅 RHEL，也不用担心 Yum 存储库！

现在，从新的容器映像启动一个容器:

```
$ sudo podman run --name hello -p 8080:8080 -d localhost/php-ubi
```

并使用 curl 测试您的容器:

```
$ curl localhost:8080
<html>
<body>
Hello, world!
</body>
</html>
```

现在，您可以在公共容器注册中心(如 Quay.io)发布容器图像。使用 UBI 时，Red Hat 允许您这样做。

在 Quay.io 个人帐户上创建了 *php-ubi* 存储库之后，您将能够:

```
$ sudo podman login -u "*youraccount*" quay.io
Password:
Login Succeeded!
$ sudo podman tag localhost/php-ubi quay.io/*youraccount*/php-ubi
$ sudo podman push quay.io/*youraccount*/php-ubi
...
Writing manifest to image destination
Storing signatures
```

你可以在 quay.io/flozanorht/php-ubi 找到我的。

## 最小是多少？

作为一个实验，让我们改变 Dockerfile 来使用 *ubi-minimal* 容器映像。下面的清单突出了这些变化:

```
FROM registry.redhat.io/ubi7/ubi-minimal

RUN microdnf -y install --nodocs \
  httpd24 rh-php72 rh-php72-php \
  && microdnf clean all

ADD index.php /opt/rh/httpd24/root/var/www/html

RUN sed -i 's/Listen 80/Listen 8080/' \
  /opt/rh/httpd24/root/etc/httpd/conf/httpd.conf \
  && chgrp -R 0 /var/log/httpd24 /opt/rh/httpd24/root/var/run/httpd \
  && chmod -R g=u /var/log/httpd24 /opt/rh/httpd24/root/var/run/httpd

EXPOSE 8080

USER 1001

CMD scl enable httpd24 rh-php72 -- httpd -D FOREGROUND
```

构建一个新的容器映像，并将其大小与之前的进行比较:

```
$ sudo podman build -t php-ubi-minimal .
$ sudo podman images
REPOSITORY                           TAG     IMAGE ID       CREATED          SIZE
localhost/php-ubi-minimal            latest  ab3c0bd38e3f   22 seconds ago   270 MB
localhost/php-ubi                    latest  5407a95fdd01   30 minutes ago   280 MB
...
registry.redhat.io/ubi7/ubi-minimal  latest  8d0998e077d3   4 weeks ago       83 MB
registry.redhat.io/ubi7/ubi          latest  c096c0dc7247   4 weeks ago      214 MB
```

看到 *ubi* 和 *ubi-minimal* 在大小上有相当大的差异(214MB 对 83MB)，但这种差异并没有转化为我们的 PHP 应用程序映像(280MB 对 270MB)。这种差异是由 Apache HTTPd、PHP 和 SCL 所需的所有依赖项造成的。我可以通过仔细选择要安装的包来修整我的形象，但是，最终，UBI 的选择可能不会对您的应用程序产生巨大的影响。

经验教训:除了基本映像大小之外，还有更多因素会使您的应用程序映像变小或变大。我宁愿从一个可靠的基于 RHEL 的 UBI 映像开始，而不是使用其他发行版的映像。另一个基本映像会像 Red Hat 一样提供错误和安全修复以及这些修复的反向移植吗？

## UBI8 呢？

将我的 Dockerfile 文件改为使用基于 RHEL8 的 UBI 并不困难。RHEL8 使用 [AppStreams](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/) 代替 SCL，其 PHP 实现只提供 FastCGI。这意味着我需要在启动 Apache HTTPd 之前启动 *php-fpm* 守护进程。

这是 *ubi8* 的完整文档:

```
FROM registry.redhat.io/ubi8/ubi

RUN yum --disableplugin=subscription-manager -y module enable \
 php:7.2 \
  && yum --disableplugin=subscription-manager -y install \
  httpd php \
  && yum --disableplugin=subscription-manager clean all

ADD index.php /var/www/html

RUN sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf \
  && mkdir /run/php-fpm \
  && chgrp -R 0 /var/log/httpd /var/run/httpd /run/php-fpm \
  && chmod -R g=u /var/log/httpd /var/run/httpd /run/php-fpm

EXPOSE 8080

USER 1001

CMD php-fpm & httpd -D FOREGROUND
```

您可以使用上面相同的 Podman 命令来测试这个 Dockerfile 文件。我在 GitHub 的项目也提供了一个 *ubi8-minimal* 变体，我相信你可以自己创建。

出于好奇，以下是标准 ubi8 图像和最小 ubi 8 图像的尺寸差异:

```
$ sudo podman images
REPOSITORY                            TAG     IMAGE ID       CREATED         SIZE
localhost/php-ubi-minimal             latest  ab87af70662a   2 minutes ago   181 MB
localhost/php-ubi                     latest  aed784c69302   12 minutes ago  264 MB
...
registry.redhat.io/ubi8/ubi           latest  4a0518848c7a   3 weeks ago     216 MB
registry.redhat.io/ubi8/ubi-minimal   latest  3bfa511b67f8   3 weeks ago      91.7 MB
```

使用 *ubi8* 父映像时，我的应用程序映像的大小差异要大得多。这说明 AppStreams 打包比 SCL 打包更有效率。尽管 *ubi8* 容器图像比 *ubi7* 容器图像稍大，但最终的最小应用程序图像要小得多。

### 了解更多信息

*   [GitHub 上的示例代码](https://github.com/flozanorht/testphp-ubi)
*   [红帽集装箱目录](https://access.redhat.com/containers)
*   [红帽软件收藏](https://developers.redhat.com/products/softwarecollections/overview/)
*   [介绍红帽通用基础图像](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)
*   [红帽容器目录上的所有 UBI 图像](https://access.redhat.com/containers/#/search/UBI)
*   [在 RHEL 引入应用流 8](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)
*   [码头工人的波德曼和 Buildah】](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)
*   [波德曼基础知识备忘单](https://developers.redhat.com/blog/2019/04/25/podman-basics-cheat-sheet/)

**6 月 13 日更新:**增加说明解释使用【registry.access.redhat.com】(公共注册表)作为 *registry.redhat.io* (私有注册表)的 al 替代。

*Last updated: September 23, 2019*