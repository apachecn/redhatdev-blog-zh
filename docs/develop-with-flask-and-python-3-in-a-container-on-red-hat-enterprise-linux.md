# 在 Red Hat Enterprise Linux 的一个容器中使用 Flask 和 Python 3 进行开发

> 原文：<https://developers.redhat.com/blog/2019/09/12/develop-with-flask-and-python-3-in-a-container-on-red-hat-enterprise-linux>

在我之前的文章[在 RHEL 7](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/) 上的容器中运行红帽企业 Linux 8 中，我展示了如何开始使用红帽企业 Linux 8 可用的最新版本的语言、数据库和 web 服务器进行开发，即使您仍然运行 RHEL 7。在本文中，我将在此基础上展示如何使用 Python 3 的当前 RHEL 8 应用程序流版本开始使用 Flask 微框架。

在我看来，在容器中使用 Red Hat Enterprise Linux 8 应用程序流比在 RHEL 7 上使用软件集合更好。虽然您需要熟悉容器，但是所有的软件都安装在您期望的位置。不需要使用`scl`命令来管理选择的软件版本。相反，每个容器都有一个独立的用户空间。你不必担心版本冲突。

在本文中，您将使用 Buildah 创建一个 Red Hat Enterprise Linux 8 Django 容器，并使用 Podman 运行它。代码将存储在您的本地机器上，并在运行时映射到容器中。您将能够像编辑任何其他应用程序一样在本地机器上编辑代码。因为它是通过卷挂载映射的，所以您对代码所做的更改将立即从容器中可见，这对于不需要编译的动态语言来说很方便。虽然这种方法不是为生产而做事情的方式，但是您会得到与在没有容器的本地开发时相同的开发内循环。本文还展示了如何使用 Buildah 为您完成的应用程序构建一个产品映像。

另外，您将在一个由`systemd`管理的容器中设置 Red Hat Enterprise Linux 8 PostgreSQL 应用程序流。您可以使用`systemctl`来启动和停止容器，就像您对非容器安装一样。

## 在 Red Hat Enterprise Linux 7 上安装 Podman 和 Buildah

首先，我们需要安装 Podman，它在红帽企业版 Linux 7 上的`extras` repo 中。`extras`回购默认不启用。开发者还应该启用`rhscl` ( [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview))、`devtools`和`optional`回购:

```
$ sudo subscription-manager repos --enable rhel-7-server-extras-rpms \
    --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms
```

现在安装波德曼和 Buildah。如果您的系统上没有安装`sudo`，请参见[如何在 Red Hat Enterprise Linux](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/) 上启用`sudo`。

```
$ sudo yum install podman buildah
```

稍后，我们将运行带有 `systemd` 的容器。如果您的系统启用了 SELinux(默认情况下)，您必须打开`container_manage_cgroup`布尔值来运行带有 `systemd` 的容器:

```
$ sudo setsebool -P container_manage_cgroup on
```

有关更多信息，请参见运行`systemd` 的[容器解决方案。](https://access.redhat.com/solutions/3387631)

**注意:**您加入 Red Hat Developer 时创建的 Red Hat ID 允许您访问 [Red Hat 客户门户网站上的内容](https://access.redhat.com)。

## 建立一个烧瓶示例应用程序

我们需要烧瓶代码来运行。让我们使用 Flask 发行版的`examples/tutorial`目录中的示例应用程序 Flaskr。将 Flask 下载到主机上的工作目录中，并提取教程应用程序:

```
$ sudo mkdir /opt/src
$ sudo chown $USER:$USER /opt/src
$ cd /opt/src
$ mkdir flask-app
$ curl -L https://github.com/pallets/flask/archive/1.1.1.tar.gz | tar xvzf - 
$ cp -pr flask-1.1.1/examples/tutorial flask-app
```

我们现在在`/opt/src/flask-app`已经有了一个 Flask 应用的例子。

## 在 Red Hat Enterprise Linux 8 容器中运行 Python 3.6 和 Flask(手动)

现在我们需要 Python 3.6 和 Flask。我们将手动设置一个包含依赖项的容器，然后运行应用程序来看看它是如何完成的。先说红帽企业版 Linux 8 通用基础映像(UBI)。如果你不熟悉 RHEL UBIs，请参阅“红帽通用基础图像”一节

Red Hat 有一个新的容器注册表，它使用了认证: [`registry.redhat.io`](http://registry.redhat.io/) 。红帽账户不需要使用 UBI 图像，但是其他不属于 UBI 的红帽图像只能通过`registry.redhat.io`获得。您加入 Red Hat Developer 时创建的 Red Hat ID 允许您访问 Red Hat 容器注册表，因此为了简单起见，我在本例中只使用了`registry.redhat.io`。

如果您在尝试提取映像时没有登录，您会收到一条详细的错误消息:

```
...unable to retrieve auth token: invalid username/password.
```

使用您的 Red Hat 用户名和密码登录:

```
$ sudo podman login registry.redhat.io
```

**注意:**波德曼被设计成无根运行。然而，Red Hat Enterprise Linux 7.6 不支持这一特性。更多信息，请参见 Scott McCarty 的，[在 RHEL 7.6](https://www.redhat.com/en/blog/preview-running-containers-without-root-rhel-76) 中运行无根容器的预览。

现在运行容器，使我们的源目录`/opt/src`在容器内可用，并暴露端口 5000，这样您就可以通过主机系统上的浏览器连接到 Flask 应用程序:

```
$ sudo podman run -v /opt/src:/opt/src:Z -it -p 5000:5000 registry.redhat.io/ubi8/ubi /bin/bash
```

前面的命令还调用了基于 Red Hat Enterprise Linux 8 的 UBI 容器的交互式 shell。从容器内部，查看哪些应用流可用于 RHEL 8:

```
# yum module list
```

您可能会注意到一组额外的应用程序流，标记为通用基础映像。有关 Red Hat Universal 基本映像的更多信息，请参见 UBI 部分。

接下来，安装 Python 3.6:

```
# yum -y module install python36
```

Python 3.6 现在已经安装在我们的容器中，并作为`python3`而不是`python`位于我们的路径中。如果你想知道为什么要看 Petr Viktorin 的文章， [Python 在 RHEL 8](https://developers.redhat.com/blog/2018/11/14/python-in-rhel-8-3) 。

接下来，使用`pip`安装烧瓶:

```
# pip3 install flask
```

您将得到一个关于以 root 用户身份运行`pip`的警告。在真实系统上以 root 身份运行`pip`通常不是一个好主意。然而，我们运行在一个隔离的一次性专用容器中，所以我们可以对`/usr`中的文件做任何我们想做的事情。

让我们检查一下 Flask 命令行界面(CLI)的安装位置:

```
# which flask
```

Pip 将其安装到`/usr/local/bin`中。

现在让我们在容器内部运行示例应用程序:

```
# cd /opt/src/flask-app
# export FLASK_APP=flaskr
# export FLASK_ENV=development
# flask init-db
# flask run --host=0.0.0.0
```

使用主机系统上的浏览器，转到`http://localhost:5000/`并查看结果页面:

[![](img/eed7143a3496cadc62abaff8203788f8.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/08/flask-flaskr-screenshot.png)

现在，您已经有了一个手动配置的容器，它使用 Red Hat Enterprise Linux 8 的 Python 3.6 应用程序流在您的 RHEL 7 系统上运行 Flask 应用程序。您可以像对待“宠物”一样对待这个容器，当您想再次运行它时使用`podman restart -l`和`podman attach -l`——只要您不删除它。我们没有命名容器，但是`-l`方便地选择了最后一个运行的容器。或者，您需要使用`podman ps -a`来获得 ID，或者随机生成的名称传递给`podman restart`和`podman attach`。

当您重新启动容器时，这类似于重新启动系统。安装的文件在那里，但是任何其他类似运行时状态的环境变量设置不会持久。你在大多数教程中看到的容器的生命周期是“运行然后删除”,因为容器被设计成短暂的。然而，当您需要试验时，知道如何创建和重启容器会很方便。

## 用 buildhr 创建一个烧瓶容器图像

为了使事情更简单，我们将创建一个安装了 Flask 的容器映像，并在容器运行时启动 Flask 应用程序。容器没有应用程序的副本，我们仍然将应用程序从主机系统映射到容器中。代码将存储在您的本地机器上，您可以像编辑任何其他应用程序源一样编辑它。因为它是通过卷挂载来映射的，所以您对代码所做的更改将在容器内部立即可见。

使用 Buildah 创建图像时，可以使用 Dockerfiles 或 Buildah 命令行。对于本文，我们将使用 Dockerfile 方法，因为您可能以前在其他教程中见过它。

因为我们正在处理您的主机系统和容器之间共享的文件，所以我们将使用与您的常规帐户相同的数字用户 ID (UID)来运行容器。在容器内部，在源目录中创建的任何文件都归您在主机系统上的用户 ID 所有。用`id`命令找出你的 UID:

```
$ id
```

记下该行开头的`UID=`和`GID=`后的数字。在我的系统上，我的 UID 和 GID 都是 1000。在 other 文件和这里的其他例子中，修改`USER`行以匹配您的 UID:GID。

在`/opt/src/flask-app`中，用以下内容创建`Dockerfile`:

```
FROM registry.redhat.io/ubi8/python-36

RUN pip3 install flask

# set default flask app and environment
ENV FLASK_APP flaskr
ENV FLASK_ENV development

# This is primarily a reminder that we need access to port 5000
EXPOSE 5000

# Change this to UID that matches your username on the host
# Note: RUN commands before this line will execute as root in the container
# RUN commands after will execute under this non-privileged UID
USER 1000

# Default cmd when container is started
# Create the database if it doesn't exist, then run the app
# Use --host to make Flask listen on all networks inside the container
CMD [ -f ../var/flaskr-instance/flaskr.sqlite ] || flask init-db ; flask run --host=0.0.0.0

```

Dockerfile 上的一个注释:我没有安装 Python 3.6，而是使用了 Red Hat 的 UBI 映像，该映像已经在 UBI 8 映像上安装了 Python 3.6。容器启动时运行的命令将创建数据库(如果它不存在)，然后运行 Flask 应用程序。

接下来，构建烧瓶容器(不要忘记后面的`.`):

```
$ sudo buildah bud -t myorg/myflaskapp .
```

现在我们可以运行包含我们的应用程序的 Flask 容器:

```
$ sudo podman run --rm -it -p 5000:5000 -v /opt/src/flask-app:/opt/app-root/src:Z myorg/myflaskapp
```

Flaskr 应用程序现在应该正在运行，您可以通过使用主机系统上的浏览器并转到`http://localhost:8000/`查看结果页面来验证这一点。

您现在可以像编辑任何常规源代码一样编辑`/opt/src/flask-app`中的代码。当你需要重启 Flask 的时候，Ctrl+C 容器。注意`run`命令中的`--rm`，它在容器退出时自动移除容器。

要再次启动容器，您需要再次使用上面的`podman run`命令，这将创建一个全新的容器，以及一个空无一物的新数据库。在许多情况下，这种新的开始是可取的。

## 在容器之间持久化 SQLite 数据库

Flaskr 示例使用一个 SQLite 数据库，它存储在容器中。容器是短暂的，所以当容器被删除时，在容器内所做的任何改变都将丢失。

有几种方法可以让数据库(或其他文件)在不同的运行中不受容器的影响。如上所述，您可以尝试保留容器并重启它，而不是每次都用`run`重新创建它。虽然这种实践对于试验和调试来说很方便，但这并不是实现持久性的好方法。现在是时候提一下了，如果你*确实*改变了文件，你想从一个已经退出但还没有被移除的容器中取出，Podman 和 Buildah 有一个方便的`mount`命令，可以在主机系统上挂载容器，这样你就可以通过文件系统访问文件。

**注:**如果你对容器和容器图像的区别感到困惑，请参阅 Scott McCarty 的文章:[容器术语实用介绍](https://developers.redhat.com/blog/2018/02/22/container-terminology-practical-introduction/)。

一个更干净的解决方案是将数据库(或您希望持久化的其他文件)存储在主机的文件系统中，而不是试图保留容器。您可以通过在`run`命令中添加另一个带有`-v`的卷挂载来实现这一点。下面是完整的命令，它存储了带有源代码的数据库:

```
$ sudo podman run --rm -it -p 5000:5000 -v /opt/src/flask-app:/opt/app-root/src:Z \
    -v /opt/src/flask-app/instance:/opt/app-root/var/flaskr-instance:Z myorg/myflaskapp
```

## 在容器中运行 MariaDB

处理持久性的另一种方法是在另一个容器中运行数据库服务器。在之前的文章中，[在 RHEL 7](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/) 的容器中运行红帽企业 Linux 8，我展示了如何在 RHEL 7 系统上使用当前的红帽企业 Linux 8 应用程序流运行 MariaDB。MariaDB 容器由`systemd`管理，所以您可以像使用非容器化版本一样使用`systemctl`命令。

为了简洁起见，我不会在本文中重复运行 MariaDB 的指令，只需按照上一篇文章的 MariaDB 部分运行该数据库。

您需要知道的一件事是如何让您的 Flask 容器连接到数据库容器。默认情况下，容器被设计为在一个隔离的虚拟网络中运行。需要采取步骤将容器联网在一起。我认为对于本文中的场景——您只想运行几个容器——最简单的方法是安排容器共享主机的网络。

要使用主机的网络，为 Flask 和数据库容器的`run`命令添加`--net host`。如果您使用的是主机网络，则不需要选择要暴露的端口。因此，烧瓶容器的完整的`run`命令是:

```
$ sudo podman run --rm -it --net host -v /opt/src/flask-app:/opt/app-root/src:Z \
    -v /opt/src/flask-app/instance:/opt/app-root/var/flaskr-instance:Z myorg/myflaskapp
```

虽然使用主机的网络对于开发来说既快又容易，但是如果有许多 MariaDB 容器都想使用端口 3306，就会遇到端口冲突。改进这种设置的一种方法是使用 Podman 的 pod 功能，将应用程序和数据库容器放在同一个 pod 中，它们共享名称空间。参见 Brent Baude 的文章， [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)。

## 使用 buildhr 创建一个带有 flask 应用程序的图像

开发完应用程序后，您可以使用 Buildah 为 Flask 应用程序创建一个可分发的容器映像。我们将使用 Buildah 命令行，而不是 Dockerfile 文件。这种方法对于复杂的构建和自动化来说更加灵活:您可以使用 shell 脚本或任何其他用于构建环境的工具。

在`/opt/src/flask-app`中，用以下内容创建`app-image-build.sh`:

```
#!/bin/sh
# Build our Flask app and all the dependencies into a container image
# Note: OOTB on RHEL 7.6 this needs to be run as root.

MYIMAGE=myorg/myflaskapp
FLASK_APP=flaskr
FLASK_ENV=development
USERID=1000

IMAGEID=$(buildah from ubi8/python-36)
buildah run $IMAGEID pip3 install flask

buildah config --env FLASK_APP=$FLASK_APP --env FLASK_ENV=$FLASK_ENV $IMAGEID

# any build steps above this line run as root inside the container
# any steps after run as $USERID
buildah config --user $USERID:$USERID $IMAGEID

buildah copy $IMAGEID . /opt/app-root/src
buildah config --cmd '/bin/sh run-app.sh' $IMAGEID

buildah commit $IMAGEID $MYIMAGE

```

这个图像调用一个启动脚本来启动我们的应用程序。接下来，在同一个目录下创建`run-app.sh`，内容如下:

```
#!/bin/sh

APP_DB_PATH=${APP_DB_PATH:-../var/instance/flaskr.sqlite}

if [ ! -f ${APP_DB_PATH} ]; then
echo Creating database
flask init-db
fi

echo Running app $FLASK_APP
flask run --host=0.0.0.0

```

现在，构建图像:

```
$ sudo app-image-build.sh
```

运行并测试新映像:

```
$ sudo podman run --rm -it --net host -v /opt/src/flask-app/instance:/opt/app-root/var/flaskr-instance:Z myorg/myflaskapp
```

当你准备好了，你可以把你的应用程序推送到一个容器注册中心，比如 Red Hat 的 [Quay.io](https://quay.io/) 。

## 后续步骤

到目前为止，您应该看到在容器中运行所需的软件组件很容易，因此您可以专注于开发。这应该和没有容器的开发没有太大的不同。

您构建的 Flask 容器没有绑定到特定的应用程序。您可以通过覆盖环境变量来为其他 Flask 应用程序重用该容器:将`-e FLASK_APP mynewapp`添加到`podman run`命令中。

您还可以在上面的 docker 文件的基础上为您的应用程序安装更多的 Python 模块到您的容器映像中，或者自定义应用程序的启动方式。

查看红帽容器目录中还有哪些 UBI 8 图像。如果语言、运行时或服务器不能作为 UBI 映像使用，您可以用`ubi8`基础映像构建自己的起点。然后，您可以使用 other 文件中的`yum`命令或`buildah run`添加应用流和其他所需的 rpm。

## 红帽通用基本图像

我在本文中多次提到通用基础映像(UBIs ),但没有解释它们。Red Hat 提供这些 ubi 作为容器图像的基础。来自 Mike Guerette 的文章， [Red Hat Universal Base Image:它如何在 3 分钟或更短时间内工作](https://developers.redhat.com/blog/2019/07/29/red-hat-universal-base-image-how-it-works-in-3-minutes-or-less/):

> “Red Hat Universal Base Images (UBI)是符合 OCI 标准的基于容器的操作系统映像，带有补充的运行时语言和可自由再分发的软件包。像以前的 RHEL 基础映像一样，它们是从 Red Hat Enterprise Linux 的一部分构建的。UBI 映像可以从 Red Hat 容器目录中获得，并且可以在任何地方构建和部署。
> 
> “而且，你不需要成为一个红帽客户来使用或重新分发它们。真的。”

随着 Red Hat Enterprise Linux 8 在 5 月份的发布，Red Hat [宣布](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)所有 RHEL 8 基本映像将在新的[通用基本映像最终用户许可协议(EULA)](https://www.redhat.com/licenses/EULA_Red_Hat_Universal_Base_Image_English_20190422.pdf) 下可用。这一事实意味着您可以构建和重新分发使用 Red Hat 的 UBI 映像作为基础的容器映像，而不是切换到基于其他发行版的映像，如 Alpine。换句话说，在构建容器时，您不必从使用`yum`切换到使用`apt-get`。

Red Hat Enterprise Linux 8 有三个基本映像。标准的叫做`ubi`，或者更准确的说是`ubi8/ubi`。这是上面使用的图片，您可能会经常使用。另外两个是最小容器。当图像大小是一个高优先级时，它们包含很少的支持软件，以及允许您在由`systemd`管理的容器内运行多个进程的多服务图像。

**注意:**如果您想构建和分发在 RHEL 7 映像上运行的容器，在`ubi7`下也有适用于 Red Hat Enterprise Linux 7 的 UBI 映像。对于本文，我们将只使用`ubi8`图片。

如果你刚刚开始使用容器，你现在不需要钻研 UBI 的细节。只需使用`ubi8`映像来构建基于 Red Hat Enterprise Linux 8 的容器。但是，当您开始分发容器映像或有关于支持的问题时，您会希望了解 UBI 的详细信息。有关更多信息，请参阅本文末尾的参考资料。

### 更多信息

相关文章:

*   [在 RHEL 7 上的容器中运行红帽企业 Linux 8](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/)(涵盖了在容器中运行的 PHP 7.2、MariaDB、WordPress)
*   [在 RHEL 8 测试版上设置 Django 应用](https://www.redhat.com/en/blog/setting-django-application-rhel-8-beta)

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
*   [红帽环球基地图片(UBI)](https://developers.redhat.com/products/rhel/ubi/)
*   [UBI 常见问题](https://developers.redhat.com/articles/ubi-faq/)

*Last updated: January 4, 2022*