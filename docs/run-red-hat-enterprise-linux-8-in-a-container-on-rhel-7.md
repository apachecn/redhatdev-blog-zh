# 在 RHEL 7 上的容器中运行 Red Hat Enterprise Linux 8

> 原文：<https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7>

即使你还在运行 RHEL 7，你也可以开始使用最新版本的语言、数据库和网络服务器进行开发。使用容器非常简单，即使你只经历过一两次“Hello，World”。

到本文结束时，您将拥有 PHP、MariaDB 和 Apache HTTPD 的当前 [RHEL 8 应用流](https://developers.redhat.com/blog/2018/11/15/rhel8-introducing-appstreams/)版本，它们运行在容器中，由 RHEL 7 系统上的 systemd 管理。 [Podman](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/) 让这一点变得容易，因为没有容器守护进程使事情变得复杂。我们将使用 WordPress 作为你自己的应用程序代码的占位符。

在我看来，在容器中使用 RHEL 8 应用流比使用软件集合更好。所有的软件都安装在你期望的位置。不需要使用`scl`命令来管理软件的选定版本，而是每个容器都有一个独立的文件系统。你不必担心版本冲突。你只需要习惯使用[容器](https://developers.redhat.com/topics/containers/)。

如果你没有密切关注[红帽开发者博客](https://developers.redhat.com/)，有一些新的事情你应该知道。例如，如果您正在寻找 rhel8 容器映像，您可能会惊讶地发现它们位于`ubi8`而不是`rhel8.`下。本文将这些细节汇集在一起，让您开始在容器中使用 RHEL 8 应用程序流和[通用基础映像](https://developers.redhat.com/articles/ubi-faq/) (UBI ),以便您拥有最新的开发版本。

## TL；灾难恢复—如何在 RHEL 7 上的容器中运行 RHEL 8

```
$ sudo subscription-manager repos --enable rhel-7-server-extras-rpms
$ sudo yum install podman buildah
$ sudo podman login registry.redhat.io
$ sudo podman run -it registry.redhat.io/ubi8/ubi
```

现在，在 RHEL 8 容器中，查看哪些应用程序流可用，然后安装 PHP 7.2:

```
# yum module list
# yum -y module install php/7.2 
# php -v
```

现在您已经在运行容器中有了 PHP 7.2 的应用程序流，并且可以探索其他 RHEL 8 应用程序流和 rpm。在本文的后面，我们将介绍如何让 MariaDB 和 HTTPD 在容器中运行。

## 在 RHEL 7 号上安装波德曼和 Buildah

首先，我们需要安装波德曼，它在《RHEL 7》的 *extras* 回购中。默认情况下，extras repo 不启用。建议开发人员也启用 rhscl ( [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview))、devtools 和可选的 repos。

```
$ sudo subscription-manager repos --enable rhel-7-server-extras-rpms \
    --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms
```

现在安装波德曼和 Buildah。如果您的系统上没有设置 sudo，请参见[“如何在 Red Hat Enterprise Linux 上启用 sudo](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/)”

```
$ sudo yum install podman buildah
```

稍后，我们将使用 systemd 运行容器。如果您的系统上启用了 SELinux，这是默认设置，那么您必须打开`container_manage_cgroup`布尔值来运行带有 systemd 的容器。有关更多信息，请参见[“运行 systemd](https://access.redhat.com/solutions/3387631) 的容器”解决方案。

**注意:**您加入 Red Hat Developers 时创建的 Red Hat ID 允许您访问 Red Hat 客户门户网站上的内容。

```
$ sudo setsebool -P container_manage_cgroup on 

```

## 在容器中运行 RHEL 8

接下来，我们将提取 RHEL 8 通用基础映像；但是首先，我们要登录到新的支持认证的 Red Hat 容器注册中心， [registry.redhat.io](http://registry.redhat.io) 。如果您没有登录，当您试图下载一个需要验证的图像时，您会得到一个有点神秘的错误消息。

使用您的 Red Hat 开发人员用户名和密码登录注册表:

```
$ sudo podman login registry.redhat.io
```

注意:podman 的设计使得它可以在没有 root 用户的情况下运行。然而，在 RHEL 7.6 中却没有对它的支持。有关更多信息，请参见 Scott McCarty 在 RHEL 7.6 中运行无根容器的[预览。](https://www.redhat.com/en/blog/preview-running-containers-without-root-rhel-76)

好了，现在调出 RHEL 8 号的图像:

```
$ sudo podman pull registry.redhat.io/ubi8/ubi
```

现在，您可以检查图像，以查看容器启动时将运行的命令，例如:

```
$ sudo podman inspect registry.redhat.io/ubi8/ubi | grep -A 1 Cmd
```

如果你需要关于 Podman 命令的帮助，请参见 [Podman 基础知识备忘单](https://developers.redhat.com/blog/2019/04/25/podman-basics-cheat-sheet/)。

现在运行 RHEL 8 容器:

```
$ sudo podman run -it registry.redhat.io
```

因为默认命令是`/bin/bash`，所以您将进入一个带有 RHEL 8 用户界面的 Bash shell。检查版本，然后查看可用的应用程序流:

```
# cat /etc/redhat-release
# yum module list
```

此时，您可以安装和探索应用程序流和/或其他 rpm。请记住，容器被设计成是短暂的，所以您所做的更改不会持续很久。

有关 RHEL 8 的命令，请参见 [RHEL 8 开发者备忘单](https://developers.redhat.com/blog/2019/05/07/red-hat-enterprise-linux-8-developer-cheat-sheet/)。

当你完成并退出容器后，你可以用`podman rm`来清理它。为了在开发过程中使事情变得简单，有一个选项，`-a`将删除系统中的所有容器:

```
$ sudo podman rm -a 

```

## 红帽通用基本图像

在容器中 RHEL 8 的`yum module list`输出中，您可能会注意到标记为 Red Hat Universal Base Image 8 的应用程序流和标记为 Red Hat Enterprise Linux 8 的其他应用程序流。那么，UBI 到底是什么？

来自 Mike Guerette 的文章，“[Red Hat Universal Base Image:How it work in 3 minutes or less”](https://developers.redhat.com/blog/2019/07/29/red-hat-universal-base-image-how-it-works-in-3-minutes-or-less/):

“Red Hat Universal Base Images (UBI)是符合 OCI 标准的基于容器的操作系统映像，带有补充的运行时语言和可自由再分发的软件包。像以前的 RHEL 基础映像一样，它们是从 Red Hat Enterprise Linux 的一部分构建的。UBI 映像可以从 Red Hat 容器目录中获得，并且可以在任何地方构建和部署。

而且，你不需要成为一个红帽客户来使用或重新分发它们。真的。"

随着 RHEL 8 在 5 月份的发布，Red Hat [宣布](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)所有 RHEL 8 的基本映像将在新的[通用基本映像最终用户许可协议(EULA)](https://www.redhat.com/licenses/EULA_Red_Hat_Universal_Base_Image_English_20190422.pdf) 下可用。这意味着您可以构建和重新分发使用 Red Hat 的 UBI 映像作为基础的容器映像，而不必切换到基于其他发行版的映像，如 Alpine。换句话说，在构建容器时，您不必从使用 yum 切换到使用 apt-get。

RHEL 8 有三个基本映像。标准的叫做`ubi`，或者更准确的说是`ubi8/ubi`。这是上面使用的图像，也是您可能最常用的图像。另外两个是非常小的容器；当映像大小是高优先级时，它们几乎没有支持软件，并且多服务映像允许您在 systemd 管理的容器内运行多个进程。

**注意:**如果您想构建和分发在 RHEL 7 映像上运行的容器，在`ubi7`下也有 RHEL 7 的 UBI 映像。对于本文，我们将只使用`ubi8`图像。

如果您刚刚开始使用容器，现在可能不需要深入 UBI 细节。仅仅使用`ubi8`图像来建造基于 RHEL 8 的容器。但是，当您开始分发容器映像或有关于支持的问题时，您会希望了解 UBI 的详细信息。有关更多信息，请参阅本文末尾的参考资料。

## 在容器中运行 MariaDB

在下一步中，我们将让 RHEL 8 MariaDB 应用程序流在主机系统上由 systemd 管理的容器中运行。我们将遵循 Alessandro Arrichiello 的文章[“使用 Podman](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/) 管理集装箱化系统服务”中的示例，但将其更新为使用 RHEL 8 应用流。

首先，拉下 MariaDB 图像。**注意:**撰写本文时，Red Hat 容器目录中还没有基于 UBI 的 MariaDB 映像。

```
$ sudo podman pull registry.redhat.io/rhel8/mariadb-103
```

因为容器被设计成短暂的，所以我们需要为数据库建立永久存储。我们将在主机系统上建立一个目录，并将其映射到容器中。首先，检查图像以找到目录所需的用户 ID。或者，我们也可以从 [Red Hat Container Catalog](https://access.redhat.com/articles/2959661) 页面获得关于这个图像的信息。

```
$ sudo podman inspect mariadb-103 | grep -A 1 User
```

获得用户 ID 后，在主机上创建一个目录，并赋予该用户 ID 所有权。

```
$ sudo mkdir -p /opt/var/lib/mysql/data
$ sudo chown 27:27 /opt/var/lib/mysql/data
```

接下来，创建一个 systemd 单元文件来管理 mysqld。作为 root 用户，使用编辑器或`cat >`创建包含以下内容的`/etc/systemd/system/mariadb-wordpress.service`:

```
[Unit]
Description=Custom MariaDB Podman Container
After=network.target

[Service]
Type=simple
TimeoutStartSec=5m
ExecStartPre=-/usr/bin/podman rm "mariadb-wordpress"

ExecStart=/usr/bin/podman run --name mariadb-wordpress -v /opt/var/lib/mysql/data:/var/lib/mysql/data:Z -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=mysecret -e MYSQL_DATABASE=wordpress --net host registry.redhat.io/rhel8/mariadb-103

ExecReload=-/usr/bin/podman stop "mariadb-wordpress"
ExecReload=-/usr/bin/podman rm "mariadb-wordpress"
ExecStop=-/usr/bin/podman stop "mariadb-wordpress"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

接下来，告诉 systemd 重新加载，启动 MariaDB 服务，然后检查输出:

```
$ sudo systemctl daemon-reload
$ sudo systemctl start mariadb-wordpress 
$ sudo systemctl status mariadb-wordpress
```

MariaDB 端口 3306 对主机系统公开。因此，如果您安装了客户端，您应该能够连接到数据库。

## 设置 HTTPD 和 PHP 容器

接下来，我们需要一个 web 服务器和一个运行在容器中的最新 PHP。浏览 Red Hat 容器目录，您会发现一个包含 Apache HTTPD 2.4 并基于 ubi8 的 PHP 7.2 映像。从拉下图像开始:

```
$ sudo podman pull registry.redhat.io/ubi8/php-72
```

检查图像以检查用户 ID、端口和工作目录:

```
$ sudo podman inspect registry.redhat.io/ubi8/php-72 | egrep -A 1 User\|ExposedPorts\|WorkingDir
```

接下来，创建一个 systemd 单元文件来管理 HTTPD。作为 root 用户，使用编辑器或`cat >`创建包含以下内容的`/etc/systemd/system/httpd-wordpress.service`:

```
[Unit]
Description=Custom httpd + php Podman Container for example app

After=mariadb-wordpress.service

[Service]
Type=simple
TimeoutStartSec=30s
ExecStartPre=-/usr/bin/podman rm "httpd-wordpress"

ExecStart=/usr/bin/podman run --name httpd-wordpress -p 8080:8080 -v /opt/src/wordpress:/opt/app-root/src:Z --net host registry.redhat.io/ubi8/php-72 /bin/sh -c /usr/libexec/s2i/run

ExecReload=-/usr/bin/podman stop "httpd-wordpress"
ExecReload=-/usr/bin/podman rm "httpd-wordpress"
ExecStop=-/usr/bin/podman stop "httpd-wordpress"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

告诉 systemd 重新加载，但先不要启动；我们需要首先创建我们的示例应用程序。

```
$ sudo systemctl daemon-reload
```

## 使用 WordPress 作为 PHP 应用程序的例子

接下来，我们将使用 WordPress 作为正在开发的 PHP 应用程序的占位符。代码将存储在您的本地机器上，并在运行时映射到 HTTPD/PHP 容器中。您将能够像编辑任何其他应用程序一样在本地机器上编辑代码。因为它是通过卷挂载来映射的，所以您对代码所做的更改将在容器内部立即可见，这对于不需要编译的动态语言来说非常方便。这不是为生产使用而做事情的方法，但这是一种快速开始的方法，并且应该提供与没有容器的本地开发基本相同的开发内循环。

下拉并解压最新的 WordPress 源代码:

```
$ cd /tmp
$ curl -L -o wordpress-latest.tar.gz https://wordpress.org/latest.tar.gz
```

现在我们需要用 httpd 容器可以映射和访问的代码创建一个目录。虽然 WordPress 是一个不错的典型演示应用，但是它有一点问题，因为它需要配置文件的写权限。理想情况下，配置应该与代码分开存储。为了本文简洁起见，我们将快速修改一下，将 WordPress 目录的所有权更改为 HTTPD 容器运行的用户 ID(这是我们之前获得的)。

```
$ sudo mkdir -p /opt/src/wordpress
$ sudo tar -C /opt/src -xvf /tmp/wordpress-latest.tar.gz
$ sudo chown -R 1001 /opt/src/wordpress
```

## 启动 HTTPD 和 PHP 容器，测试应用程序

用 systemd 启动容器，然后检查状态:

```
$ sudo systemctl start httpd-wordpress
$ sudo systemctl status httpd-wordpress
```

容器运行后，使用主机系统上的浏览器导航到 http://127.0.0.1:8080/。你应该会看到 WordPress“设置语言”屏幕。按照提示开始安装。然后，设置连接到数据库的参数:

*   数据库名称:wordpress
*   用户名:wordpress
*   密码:mysecret
*   数据库主机:127.0.0.1:3306
*   表格前缀:wp_

[![WordPress Database Configuration](img/ea8714ddc95625bf3bb9dbde53cafe27.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/08/wordpress-db-setup.png)

如果您想从不同的机器访问 HTTPD 服务器，您需要打开端口 8080 的防火墙:

```
$ sudo firewall-cmd --permanent --add-port=8080/tcp
$ sudo firewall-cmd --add-port=8080/tcp
```

现在，您可以在本地机器上使用 MariaDB、PHP 和 Apache HTTPD 的容器版本开发应用程序。

## 使用 buildhr 用你的 php 应用程序创建一个图像

开发完应用程序后，您可以使用 Buildah 为您的应用程序创建一个容器映像，该映像基于 UBI 8 PHP 7.2 映像。当容器映像运行时，它将引入运行在 UBI 8 基础映像之上 PHP 和 HTTPD。

使用 Buildah，您可以使用 Dockerfiles 或命令行，它们更适合构建自动化和复杂的构建。首先，Dockerfile 方法:

在`/opt/src`中创建此 Dockerfile 文件:

```
FROM registry.redhat.io/ubi8/php-72
ADD wordpress /opt/app-root/src
CMD [ "/bin/sh", "-c", "/usr/libexec/s2i/run"]
```

来构建图像(不要忘记后面的“.”):

```
$ cd /opt/src
$ sudo buildah bud -t myorg/myphpapp .
```

您可以使用 Buildah 或 Podman 检查图像:

```
$ sudo buildah inspect myorg/myphpapp
```

要运行新的映像，首先停止 systemd 容器，然后启动新的容器

```
$ sudo systemctl stop httpd-wordpress
$ sudo podman run --name myphpapp --net host -d -p 8080:8080 myorg/myphpapp
```

如果您在浏览器中访问 http://127.0.01:8080/

现在，您可以将`myorg/myapp`图像推送到容器注册表中，与其他人共享。

我们可以用 Buildah 命令行来构建图像，而不是使用 Dockerfile 文件。以下是构建相同映像的命令:

```
$ su -
# buildah --name myorg/myphpapp from ubi8/php-72
# buildah copy myorg/myphpapp wordpress /opt/app-root/src
# buildah config --cmd '/bin/sh -c /usr/libexec/s2i/run' myorg/myphpapp
# buildah commit myorg/myphpapp
```

Buildah 还有其他不错的特性，包括 mount 命令，它允许您在主机系统上挂载映像以供检查:

```
# mountpoint=$(buildah mount ${container})
# cd $mountpoint
```

现在，您可以把这个映像看作一个常规的文件系统。您也可以在构件过程中修改图像。这为简化构建图像的方式提供了许多可能性。请参见“[Buildah](https://opensource.com/article/18/6/getting-started-buildah)入门”进行概述。

## 后续步骤

到目前为止，您应该已经看到在容器中运行所需的软件组件是非常容易的，这样您就可以专注于开发了。这应该和没有容器的开发没有太大的不同。

一旦您的应用程序准备就绪，您就可以将其构建到容器映像中，该映像使用您在开发过程中使用的相同组件。你可以把它推送到一个注册表，比如 Red Hat 的 [Quay.io](https://quay.io/) 。

你可以在[红帽容器目录](https://access.redhat.com/containers/#/search/ubi8)中查看其他可用的 UBI 8 图片。如果语言、运行时或服务器不能作为 UBI 映像使用，您可以从 ubi8 基础映像开始构建自己的映像。然后，您可以使用 other 文件中的 yum 命令或者使用`buildah run.`来添加应用程序流和您需要的其他 rpm

本文中的设置有几个缺点。这是一个快速简单的演示，有很多方法可以改进设置。例如，容器被配置为与`--net host,`共享主机网络，这使得 web 服务器连接到数据库服务器变得简单。尽管这对于开发来说既快速又容易，但是您没有从默认容器网络配置中获得的网络隔离。如果有许多应用程序想要使用同一个端口，您可能会遇到端口冲突。

改进配置的一种方法是使用 Podman 的 pod 功能，将 web 和数据库容器放在同一个 pod 中，在这个 pod 中它们共享名称空间。

## 容器兼容性和可支持性的限制

如果你正在考虑在生产中在 RHEL 7 之上运行 RHEL 8 容器，你应该参考 [Red Hat Enterprise Linux 容器兼容性矩阵](https://access.redhat.com/support/policy/rhel-container-compatibility)来看看什么适合你的具体情况。

一个常见的误解是，你可以在任何容器主机上运行任何容器，并且*一切都会正常工作*。容器的一个优点(也是一个缺点)是您运行的任何容器都共享主机的 Linux 内核。为了应用程序和内核版本之间的兼容性，人们一直在努力维护应用程序二进制接口(ABI ),但随着时间的推移，也发生了一些变化，可能会以微妙的方式破坏事物。

客观地说，RHEL 8 使用 Linux 内核版本 4.18，而 RHEL 7 使用内核版本 3.10。因此，当你在 RHEL 7 上运行 RHEL 8 容器时，你运行的是在 3.10 内核上为 4.18 内核构建和测试的东西。由于 ABI 兼容性，许多东西只要不深入到那些随着时间而变化的更深奥的系统接口中，就能正常工作。正如您从大量可用的容器化软件中所看到的，大多数应用程序都可以在一系列主机系统上工作，但是不能保证所有的东西都已经过测试或者完全受支持。

有关更多细节，请参阅 Scott McCarty 的文章“容器的兼容性和可支持性的限制”他解释了可能(和已经)发生的问题。然而，他说，“在 Red Hat，我们有信心在 RHEL 7 号和 RHEL 8 号集装箱主机上提供 RHEL 6 号、RHEL 7 号和 RHEL 8 号集装箱映像的支持和修补。”希望，我没有把它扯得太远。

### 更多信息

备忘单:

*   [波德曼基础知识小抄](https://developers.redhat.com/blog/2019/04/25/podman-basics-cheat-sheet/)
*   [红帽企业版 Linux 8 小抄](https://developers.redhat.com/blog/2019/05/07/red-hat-enterprise-linux-8-developer-cheat-sheet/)

波德曼和布尔达:

*   [针对 Docker 用户的 Podman 和 Buildah](https://developers.redhat.com/blog/2019/02/21/podman-and-buildah-for-docker-users/)
*   [与搬运工一起管理集装箱化系统服务](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/)
*   [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)
*   [Buildah 入门](https://opensource.com/article/18/6/getting-started-buildah)
*   [构建、运行和管理容器——RHEL 8 文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html-single/building_running_and_managing_containers/index)
*   [集装箱入门- RHEL 7 文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html-single/getting_started_with_containers/index)

UBI:

*   [红帽万能基础图:如何在 3 分钟内完成](https://developers.redhat.com/blog/2019/07/29/red-hat-universal-base-image-how-it-works-in-3-minutes-or-less/)
*   [红帽开发者的 UBI 页面](https://developers.redhat.com/products/rhel/ubi/)
*   [UBI 常见问题](https://developers.redhat.com/articles/ubi-faq/)

*Last updated: January 5, 2022*