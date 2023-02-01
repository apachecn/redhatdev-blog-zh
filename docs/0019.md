# 使用 PHP 5.6 在 RHEL 7 上构建你的第一个应用程序，包括容器和 Red Hat 软件集合

> 原文：<https://developers.redhat.com/articles/using-php-56-rhel-7-containers-and-red-hat-software-collections>

在 15 分钟内开始在 Red Hat Enterprise Linux 上的 docker 容器中构建 PHP 5.6 应用程序。

### 简介和先决条件

在本教程中，您将学习如何在 Red Hat Enterprise Linux 上的 docker 容器中开始构建 PHP 5.6 应用程序。为了构建和运行容器，您将首先在您的 Red Hat Enterprise Linux 7 系统上安装`docker`。您将使用 Red Hat Software Collections(RH SCL)2.2 中的 PHP 5.6 容器映像作为您的容器化应用程序的基础。

您需要一个运行 Red Hat Enterprise Linux 7 Server 的系统，并且当前订阅了允许您从 Red Hat 下载软件和更新的 Red Hat。开发者可以通过[注册并通过【developers.redhat.com】的](https://developers.redhat.com/products/rhel/download/)[下载](https://developers.redhat.com/)获得一个免费的用于开发目的的红帽企业 Linux 开发者套件订阅。

如果您在任一点遇到困难，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 1.准备好你的系统

*5 分钟*

在这一步中，您将配置您的系统来构建和运行 docker 容器。在此过程中，您将添加必要的软件库，然后验证您的系统是否有最新的 Red Hat 订阅，并且能够从 Red Hat 接收更新。您的系统需要已经注册了 Red Hat。

首先，您将启用两个默认禁用的 Red Hat 软件存储库。为命令行(CLI)和图形用户界面(GUI)都提供了说明。

### 使用 Red Hat Subscription Manager GUI

红帽订阅管理器可以从*应用程序*菜单的*系统工具*组中启动。或者，您可以通过在命令提示符下键入`subscription-manager-gui`来启动它。

从 subscription manager 的*系统*菜单中选择*存储库*。在存储库列表中，检查*rhel-7-server-optional-rpms*和*rhel-7-server-extras-rpms*的【T4 启用】列。单击后，复选标记可能需要几秒钟才会出现在“已启用”列中。

### 从命令行使用 subscription-manager

您可以使用`subscription-manager`工具从命令行添加或删除软件库。如果还没有打开的话，打开一个*终端*窗口。使用`su`成为 root 用户。使用`subscription-manager --list`选项来查看可用的软件库。

```
$ su -
# subscription-manager repos --list
```

启用两个附加存储库:

```
# subscription-manager repos --enable rhel-7-server-extras-rpms
# subscription-manager repos --enable rhel-7-server-optional-rpms
```

### 安装 docker 并启动 docker 守护进程

在下一步中，您将:

1.  用任何可用的软件更新来更新您的系统

2.  使用`yum`安装`docker`和一些额外的 rpm

3.  配置`docker`守护进程在引导时启动

4.  启动`docker`守护进程

如果没有打开根*终端*窗口，启动一个*终端*窗口，用`su`成为根用户。

现在通过运行`yum update`下载并安装任何可用的更新。如果有可用的更新，`yum`将列出它们并询问是否可以继续。

```
$ su -
# yum update
```

安装`docker`和必要的附加 rpm:

```
# yum install docker device-mapper-libs device-mapper-event-libs
```

启用 docker 守护程序在引导时启动并立即启动:

```
# systemctl enable docker.service
# systemctl start docker.service
```

现在验证 docker 守护程序正在运行:

```
# systemctl status docker.service
```

您的系统现在可以构建和运行 docker 格式的容器了。如果您在任一点遇到困难，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 2.运行您的第一个容器

*5 分钟*

这一步将使用 Red Hat container registry(一个容器映像库)中的容器映像下载并安装 PHP 5.6。安装 PHP 5.6 容器映像将使 PHP 5.6 可供系统上的其他容器使用。因为容器在隔离的环境中运行，所以安装不会改变您的主机系统。您必须使用`docker`命令来使用或查看容器的内容。

本节中显示的命令可用于下载和安装其他容器，如您构建的应用程序容器。容器可以指定它们需要安装其他容器，这可以自动发生。例如，您可以在用于描述和构建您的容器的`Dockerfile`中指定您的应用程序需要 PHP 5.6。然后，当有人安装你的容器时，他们的系统会自动直接从 Red Hat 容器注册表下载所需的 PHP 5.6 容器。

PHP 5.6 容器映像是 RHSCL 的一部分，RHS cl 为 Red Hat Enterprise Linux 提供了最新的开发技术。许多 Red Hat Enterprise Linux (RHEL)订阅都包含对 RHSCL 的访问。有关哪些订阅包含 RHSCL 的更多信息，请参见[如何使用 Red Hat 软件集合(RHSCL)或 Red Hat 开发工具集(DTS)](https://access.redhat.com/solutions/472793) 。

如果没有打开根*终端*窗口，启动一个*终端*窗口，用`su`成为根用户。

要下载并安装 PHP 5.6 容器映像，请使用以下命令:

```
# docker pull registry.access.redhat.com/rhscl/php-56-rhel7
```

`docker images`命令列出了系统中的容器图像:

```
# docker images
```

该列表将包括您已经下载的那些图像和您系统上以前安装的任何容器。

现在启动一个`bash` shell 来查看使用 PHP 5.6 容器映像的容器内部。shell 提示符发生变化，这表明您正在容器内部的 shell 中键入内容。一个`ps -ef`显示容器内唯一运行的东西是`bash`和`ps`。键入`exit`离开容器的 bash shell。

```
# docker run -it rhscl/php-56-rhel7 /bin/bash
bash-4.2$ which php
/opt/rh/rh-php56/root/usr/bin/php
bash-4.2$ php -v
PHP 5.6.5 (cli) (built: Feb 16 2016 05:49:30)
bash-4.2$ pwd
/opt/app-root/src
bash-4.2$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
default      1     0  0 20:10 ?        00:00:00 /bin/bash
default     14     1  0 20:10 ?        00:00:00 ps -ef
bash-4.2$ exit
```

前面的`docker run`命令创建了一个容器来运行您的命令，保持任何状态，并将其与系统的其余部分隔离。您可以使用`docker ps`查看正在运行的容器列表。要查看所有已经创建的容器，包括那些已经退出的，使用`docker ps -a`。

您可以使用`docker start`重新启动上面创建的容器。容器是通过名称来引用的。如果你不提供名字，Docker 会自动生成一个名字。一旦容器重新启动，`docker attach`将允许您与运行在其中的 shell 交互。请参见以下示例:

```
# docker ps -a
CONTAINER ID        IMAGE                         COMMAND                CREATED              STATUS                          PORTS               NAMES
84458ca538fb        rhscl/php-56-rhel7   "container-entrypoin   About a minute ago   Exited (0) About a minute ago                       determined_mayer
# docker start determined_mayer
determined_mayer
# docker attach determined_mayer
```

此时，您已经连接到容器内部正在运行的 shell。当你连接时，你不会看到命令提示符，所以按下回车键让它打印另一个。

```
bash-4.2$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
default      1     0  0 14:44 ?        00:00:00 /bin/bash
default     11     1  0 14:45 ?        00:00:00 ps -ef
bash-4.2$ exit
```

由于容器中唯一的进程`bash`被告知`exit`，容器将不再运行。这可以用`docker ps -a`来验证。不再需要的容器可以用`docker rm *<container-name>*`清理。

```
# docker rm determined_mayer
```

要查看 Red Hat 容器注册表中还有哪些容器图像，请使用以下一个或多个搜索:

```
# docker search registry.access.redhat.com/rhscl
# docker search registry.access.redhat.com/openshift3
# docker search registry.access.redhat.com/rhel
# docker search registry.access.redhat.com/jboss
```

如果您需要帮助，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 3.在容器中构建 Hello World

*5 分钟*

在这一步中，您将创建一个使用 PHP 5.6 作为 web 服务器的小型 Hello World 容器。一旦创建，容器就可以在安装了`docker`的其他系统上运行。您需要使用您最喜欢的编辑器在一个空目录中创建几个文件，包括一个描述容器的`Dockerfile`。你不需要在根用户下运行来创建文件，但是你需要根用户权限来运行`docker`命令。

首先，创建一个空目录，然后用以下内容创建一个名为`Dockerfile`的文件，但是将`MAINTAINER`行改为包含您的姓名和电子邮件地址:

Dockerfile

```
FROM rhscl/php-56-rhel7

MAINTAINER Your Name "your-email@example.com"

EXPOSE 8000

COPY . /opt/app-root/src

CMD /bin/bash -c 'php -S 0.0.0.0:8000'
```

创建包含以下内容的文件`index.php`:

index.php

```
<?php
print "Hello, Red Hat Developers World from PHP " . PHP_VERSION . "\n";
?>
```

现在使用`docker build`构建容器映像。您需要在您创建的包含`Dockerfile`和`index.php`的目录中使用`su`或`sudo`作为 root 用户。

```
# docker build -t myname/phpweb .
```

您可以看到使用以下命令创建的容器映像:

```
# docker images
```

现在使用`docker run`运行容器。PHP 容器将创建一个小型 web 服务器，它监听容器内部的端口 8000。`run`命令将主机上的端口 8000 映射到容器内的端口 8000。

```
# docker run -d -p 8000:8000 --name helloweb myname/phpweb`
```

run 命令返回容器的唯一 ID，您可以忽略它。要检查容器是否正在运行，请使用`docker ps`。输出应该显示一个名为`helloweb`的容器，它正在运行您创建的`*myname*/phpweb`容器映像。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
c7885aa23773        myname/phpweb    "container-entrypoint"   6 seconds ago       Up 4 seconds        0.0.0.0:8000->8000/tcp, 8080/tcp   helloweb
```

使用`curl`访问 PHP web 服务器:

```
# curl http://localhost:8000/
Hello, Red Hat Developers Worldfrom PHP 5.6.5!
```

要查看运行容器中的日志，请使用`docker logs *<container-name>*`:

```
# docker logs helloweb
```

完成后，停止正在运行的容器:

```
# docker stop helloweb
```

`helloweb`容器将被保留，直到您用`docker rm`将其移除。你可以用`docker start helloweb`重启容器。注意:如果同名的容器已经存在，后续的`docker run`将产生一个错误。

您可以使用`docker inspect`查看有关容器的信息:

```
# docker inspect myname/phpweb
```

输出是一个易于阅读的 JSON 结构。 *Config* 部分包含容器运行时环境的细节，比如环境变量和默认命令。注意，容器配置中的许多信息都是从父容器继承的，在本例中是 PHP 5.6 运行时容器。

最后，当您创建的应用程序容器映像准备就绪时，您可以通过将它们推送到公共或私有容器注册中心来分发它们。然后，您的容器将可以使用`docker pull`安装到其他系统上。

#### 想知道更多关于 RHEL 的事情吗？

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**关注红帽开发者博客**
[https://developers.redhat.com/blog/](https://developers.redhat.com/blog)

## 故障排除和常见问题

1.  我的系统无法从 Red Hat 下载更新。

    您的系统必须使用`subscription-manager register`向 Red Hat 注册。你需要有一个最新的红帽订阅。

2.  **作为一名开发者，如何获得免费的红帽企业版 Linux 订阅？**

    当您通过[developers.redhat.com](https://developers.redhat.com/)注册并下载 Red Hat Enterprise Linux Server 时，一个免费的 Red Hat Enterprise Linux Developer Suite 订阅将自动添加到您的帐户中。我们建议您遵循我们的[入门指南](https://developers.redhat.com/products/rhel/get-started/)，该指南涵盖了使用您选择的 VirtualBox、VMware、Microsoft Hyper-V 或 Linux KVM/Libvirt 在物理系统或虚拟机(VM)上下载和安装 Red Hat Enterprise Linux。

3.  如何判断是否有新版本 PHP 的容器映像可用？

    如何才能看到其他可用的容器图像？

    **我找不到本教程中提到的容器，如何知道名称是否更改？**

    要查看 Red Hat 容器注册表中还有哪些容器，请使用以下一个或多个搜索:

    ```
    # docker search registry.access.redhat.com/rhscl
    # docker search registry.access.redhat.com/openshift3
    # docker search registry.access.redhat.com/rhel
    # docker search registry.access.redhat.com/jboss
    ```

4.  **找不到`docker` rpm。**

    **`yum`无法找到`docker`的转速。**

    **当我试着安装`docker`，`yum`给出错误*没有可用的包对接器*。**

    `docker` rpm 在*rhel-7-server-extras-rpms*软件库中。它仅适用于 Red Hat Enterprise Linux 的服务器版本。默认情况下，*rhel-7-server-extras-rpms*存储库是禁用的。有关启用附加软件资料库的信息，请参见本教程的第一步。

5.  从哪里可以了解更多关于使用 Linux 容器交付应用程序的信息？

    如果你还没有加入[红帽开发者计划](https://developers.redhat.com/)，请在 developers.redhat.com[注册。会员是免费的。](https://developers.redhat.com/) [容器开发的推荐实践](https://access.redhat.com/articles/1483053)和许多其他容器文章可从 [Red Hat 客户门户](https://access.redhat.com/)获得。
    如果您是 Red Hat 技术合作伙伴，请访问[Red Hat Connect for Technology Partners](http://connect.redhat.com/)网站上的[容器专区](https://access.redhat.com/articles/1483053)。

*Last updated: November 19, 2020*