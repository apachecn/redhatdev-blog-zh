# 在 Red Hat Enterprise Linux 的容器中使用 Node.js 进行开发

> 原文：<https://developers.redhat.com/blog/2019/09/13/develop-with-node-js-in-a-container-on-red-hat-enterprise-linux>

在我之前的文章[在 RHEL 7](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/) 上的容器中运行红帽企业 Linux 8 中，我展示了如何开始使用红帽企业 Linux 8 可用的最新版本的语言、数据库和 web 服务器进行开发，即使您仍然运行 RHEL 7。在本文中，我将在此基础上展示如何使用当前 RHEL 8 应用流版本的 [Node.js](https://nodejs.org/en/) 和 [Redis](https://redis.io) 5 开始使用 Node。

在我看来，在容器中使用 Red Hat Enterprise Linux 8 应用程序流比在 RHEL 7 上使用软件集合更好。虽然您需要熟悉容器，但是所有的软件都安装在您期望的位置。不需要使用`scl`命令来管理选择的软件版本。相反，每个容器都有一个独立的用户空间。你不必担心版本冲突。

在本文中，您将使用 [Buildah](https://buildah.io) 创建一个 Red Hat Enterprise Linux 8 Node.js 容器，并使用 [Podman](https://podman.io) 运行它。代码将存储在您的本地机器上，并在运行时映射到 RHEL 8 Node.js 容器中。您将能够像编辑任何其他应用程序一样在本地机器上编辑代码。因为它是通过卷挂载来映射的，所以您对代码所做的更改将立即从容器中可见，这对于不需要编译的动态语言来说很方便。这种方法并不是您想要的生产方式，但是它可以让您快速开始开发，并且应该为您提供与没有容器的本地开发基本相同的开发内循环。本文还展示了如何使用 Buildah 为您完成的应用程序构建一个映像，以便用于生产。

此外，您将在由`systemd`管理的容器中设置 Red Hat Enterprise Linux 8 Redis 应用程序流。您将能够使用`systemctl`来启动和停止容器，就像您对非容器安装一样。

## 在 Red Hat Enterprise Linux 7 上安装 Podman 和 Buildah

首先，我们需要安装 Podman，它在红帽企业版 Linux 7 上的`extras` repo 中。`extras` 回购默认不启用。建议开发者同时启用`rhscl` ( [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview))、`devtools`、`optional`回购:

```
$ sudo subscription-manager repos --enable rhel-7-server-extras-rpms \
    --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms
```

现在安装波德曼和 Buildah。如果您的系统上没有安装`sudo`，请参见[如何在 Red Hat Enterprise Linux](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/) 上启用 sudo。

```
$ sudo yum install podman buildah
```

稍后，我们将运行带有`systemd`的容器。如果在您的系统上启用了 SELinux(默认情况下)，您必须打开`container_manage_cgroup`布尔值来运行带有`systemd`的容器。有关更多信息，请参见运行 systemd 的[容器和](https://access.redhat.com/solutions/3387631)解决方案。
**注意:**您加入 Red Hat Developers 时创建的 Red Hat ID 允许您访问 [Red Hat 客户门户](https://access.redhat.com)上的内容。

```
$ sudo setsebool -P container_manage_cgroup on 

```

## 在 Red Hat Enterprise Linux 8 UBI 容器中运行 Node.js

我们将需要 Node.js 放在一个可以用于开发的容器中。我们可以删除 Red Hat Enterprise Linux 8 通用基础映像(UBI ),然后`yum install nodejs`创建我们自己的基础映像，但幸运的是，Red Hat 已经这样做了，并且可以免费使用和重新发布。有关 UBI 的更多信息，请参见下面的“Red Hat 通用基础映像”一节。

Red Hat 有一个新的使用认证的容器注册中心: [registry.redhat.io](http://registry.redhat.io/) 。使用 UBI 图像不需要 Red Hat 帐户。然而，其他不属于 UBI 的红帽图片只能通过这个注册表获得。您加入 Red Hat Developers 时创建的 Red Hat ID 允许您访问 Red Hat 容器注册表，因此为了简单起见，我只使用`registry.redhat.io`。如果您在尝试提取图像时没有登录，将会收到一条详细的错误消息。如果你仔细观察，你会发现:

```
...unable to retrieve auth token: invalid username/password.
```

使用您的 Red Hat 用户名和密码登录:

```
$ sudo podman login registry.redhat.io
```

**注意:**波德曼的设计使得它可以在没有 root 用户的情况下运行。然而，Red Hat Enterprise Linux 7.6 不支持这一功能。更多信息，请参见 Scott McCarty 的[在 RHEL 7.6](https://www.redhat.com/en/blog/preview-running-containers-without-root-rhel-76) 中运行无根容器的预览。

要查看哪些 Node.js 容器映像可用，您可以搜索 [Red Hat 容器目录](https://access.redhat.com/containers/)，或者使用命令行界面(CLI)进行搜索:

```
$ sudo podman search registry.redhat.io/ubi8
```

撰写本文时，当前的应用程序流版本是`nodejs-10`。将映像下载到您的本地系统:

```
$ sudo podman pull registry.redhat.io/ubi8/nodejs-10
```

## 设置 Node.js 示例应用程序

我们有一个安装了 Node.js 的容器，但是我们需要代码来运行。我们将使用 React.js 创建快速“Hello，World”的代码，该代码将在容器中运行，但可以从主机系统上的浏览器访问。

为了便于开发，我们不会将代码复制到容器中。相反，我们将把源目录从主机系统映射到容器中。

因为我们使用的是在您的主机系统和容器之间共享的文件，所以我们将使用与您在主机系统上的帐户相同的数字用户 ID (UID)来运行容器。如果在容器中运行的东西在源目录中创建了文件，它们将归您的用户 ID 所有。使用`id`命令找出您的 UID 和 GID:

```
$ id
```

记下该行最开始的`UID=`和`GID=`后的数字。在我的系统上，我的 UID 和 GID 都是 1000，因此您将看到这些值反映在本文中的 Podman 和 Buildah 命令中。更改这些值以匹配您的 UID 和 GID。

运行以下命令，在方便的位置创建一个源目录，以便与容器共享:

```
$ sudo mkdir -p /opt/src/
$ sudo chown -R $USER:$USER /opt/src
```

## 在容器中创建 React 应用程序

我们将使用一个`npx`命令来创建示例应用程序。当前版本的`node`、`npm`和`npx`安装在容器中，所以我们需要在容器中工作。为此，启动一个运行`nodejs`映像的容器:

```
$ sudo podman run --rm -it --user 1000:1000 -v /opt/src:/opt/app-root/src:Z --net host registry.redhat.io/ubi8/nodejs-10 /bin/bash
```

让我们看看上面的命令做了什么。它:

*   安排容器在退出时被删除。
*   将容器设置为交互式，在前台运行。
*   将容器中的进程设置为以 UID 1000 和 GID 1000 运行。
*   将主机系统的`/opt/src`目录作为`/opt/app-root/src`映射到容器中，让容器能够访问我们的源目录。
*   将容器设置为共享主机网络。(此操作使应用程序在容器中使用的任何端口都可以从主机系统访问。)
*   在容器中运行交互式 bash shell。

现在，使用容器内的 bash shell 运行这些命令:

```
$ npx create-react-app react-web-app
$ cd react-web-app
$ npm start
```

此时，新创建的 React 应用程序正在容器内部运行。使用主机系统上的浏览器，转到`http://localhost:3000/`。你会看到:

[![](img/9f00f6771c5c1fbf90d52558313fdfef.png)](https://developers.redhat.com/blog/wp-content/uploads/2019/08/Node-react-screenshot.png)

让集装箱保持运转。在主机系统上，转到`/opt/src/react-web-app/src`。然后，用编辑器编辑`App.js`。当您保存文件时，在容器中运行的 Node.js 应用程序会注意到并告诉在您的主机系统上运行的浏览器重新加载页面。现在，您应该能够以与不使用容器时几乎相同的方式开发应用程序。

## 在容器中运行 Redis

在本节中，我们将让 Red Hat Enterprise Linux 8 Redis 应用程序流在由主机系统上的`systemd`管理的容器中运行。搜索 Red Hat 容器目录，我们可以寻找 Redis 图像。在撰写本文时，Red Hat 容器目录中还没有基于 UBI 8 的 Redis 映像，但是有一个基于 RHEL 8 的。为了便于检查，我们将在运行前提取图像:

```
$ sudo podman pull registry.redhat.io/rhel8/redis-5
```

因为容器被设计成短暂的，所以我们需要为 Redis 数据存储设置永久存储。我们将在主机系统上建立一个目录，并将其映射到容器中。首先，检查图像，找出目录所需的用户 ID:

```
$ sudo podman inspect redis-5 | grep -A 1 User
```

或者，我们也可以从 [Red Hat Container Catalog](https://access.redhat.com/containers/#/registry.access.redhat.com/rhel8/redis-5) 页面获得关于这个图像的信息。

现在我们已经获得了容器将在(1001)下运行的数字用户 ID，在主机上创建一个目录，赋予该用户 ID 所有权，并为 SELinux 设置上下文:

```
$ sudo mkdir -p /opt/dbdata/node-redis-db
$ sudo chown 1001:1001 /opt/dbdata/node-redis-db
$ sudo setfacl -m u:1001:-wx /opt/dbdata/node-redis-db
$ sudo semanage fcontext -a -t container_file_t /opt/dbdata/node-redis-db
$ sudo restorecon -v /opt/dbdata/node-redis-db
```

让我们手动测试 Redis:

```
$ sudo podman run -it --name node-redis-db -p 6379:6379 -v /opt/dbdata/node-redis-db:/var/lib/redis/data:Z registry.redhat.io/rhel8/redis-5
```

您应该会看到如下所示的输出:

```
---> 22:00:01     Processing Redis configuration files ...
---> 22:00:01     WARNING: setting REDIS_PASSWORD is recommended
---> 22:00:01     Sourcing post-init.sh ...
---> 22:00:01     Cleaning up environment variable REDIS_PASSWORD ...
---> 22:00:01     Running final exec -- Only Redis logs after this point
1:C 26 Aug 2019 22:00:01.568 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 26 Aug 2019 22:00:01.568 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 26 Aug 2019 22:00:01.568 # Configuration loaded
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 5.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 1
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

```

Redis 启动正确，所以可以使用 Ctrl+C 来停止容器。通过移除容器进行清理:

```
$ sudo podman rm node-redis-db
```

接下来，创建一个`systemd`单元文件来管理 Redis。作为 root 用户，使用编辑器或`cat >`创建`/etc/systemd/system/node-redis-db.service`，内容如下:

```
[Unit]
Description=Node app Redis Database
After=network.target

[Service]
Type=simple
TimeoutStartSec=5m
ExecStartPre=-/usr/bin/podman rm "node-redis-db"

ExecStart=/usr/bin/podman run -it --name node-redis-db -e REDIS_PASSWORD=mysecret -p 6379:6379 -v /opt/dbdata/node-redis-db:/var/lib/redis/data:Z registry.redhat.io/rhel8/redis-5

ExecReload=-/usr/bin/podman stop "node-redis-db"
ExecReload=-/usr/bin/podman rm "node-redis-db"
ExecStop=-/usr/bin/podman stop "node-redis-db"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target

```

接下来，告诉`systemd`重新加载，启动 Redis 服务，然后检查输出:

```
$ sudo systemctl daemon-reload
$ sudo systemctl start node-redis-db
$ sudo systemctl status node-redis-db
```

您可以使用以下命令检查容器日志:

```
$ sudo podman logs node-redis-db
```

Redis 端口 6379 对主机系统是公开的，因此如果安装了 Redis 客户机，应该能够连接到 Redis 服务器。

关于`systemd`单元文件中的`podman run`命令，有几点需要注意。不要像从命令行那样使用`-d`选项来脱离正在运行的容器。因为`systemd`正在管理进程，所以`podman run`不应该退出，直到容器内的进程终止。如果使用`-d`，`systemd`会认为容器有故障，会重启。

没有使用`podman run`的`--rm`选项，该选项在容器退出时自动移除容器。相反，`systemd`被配置为在启动容器之前运行一个`podman rm`命令。该设置使您有机会在停止的容器退出后检查其内部文件的状态。

## 从 Node.js 容器测试 Redis

现在 Redis 正在运行，我们将从 Node.js 容器测试它。使用主机系统上的编辑器，创建包含以下内容的`/opt/src/react-web-app/redis-test.js`:

```
let redis     = require('redis'),

client    = redis.createClient({
    port      : 6379,
    host      : '127.0.0.1',
    password  : 'mysecret',
});

count = client.incr('view-count', function(err) {
  if (err) {
    throw err; /* in production, handle errors more gracefully */
  } else {
    client.get('view-count',function(err,value) {
      if (err) {
        throw err;
      } else {
        console.log(value);
        process.exit();
      }
    }
  );
};
});

```

我们需要从 Node.js 容器内部运行测试。运行以下命令启动一个:

```
$ sudo podman run --rm -it --user 1000:1000 -v /opt/src/react-web-app:/opt/app-root/src:Z --net host registry.redhat.io/ubi8/nodejs-10 /bin/bash
```

现在，使用容器内的 bash shell，安装 Redis 支持:

```
$ npm install redis
```

您现在可以运行测试应用程序:

```
$ node redis-test.js
```

每次运行测试应用程序时，计数器都会增加。现在可以为 Node.js 应用程序创建一个后端，使用 Redis 进行存储。您可以使用`systemctl restart node-redis-db`来重新启动 Redis 容器，以验证 Redis 数据存储在容器重新启动时是否被保留。

## 使用 Buildah 通过 Node.js 应用程序创建图像

开发完应用程序后，可以使用 Buildah 通过 Node.js 应用程序创建一个可分发的容器映像。虽然 Buildah 可以使用 Dockerfile 文件，但我们将使用 Buildah 命令行。对于复杂的构建和自动化来说，这个选项更加灵活。您可以使用 shell 脚本或任何用于构建环境的工具。

在`/opt/src/react-web-app`中，用以下内容创建`app-image-build.sh`:

```
#!/bin/sh
# Build our Node.js app and all the dependencies into a container image
# Note: OOTB on RHEL 7.6 this needs to be run as root.

MYIMAGE=myorg/mynodeapp

USERID=1000

IMAGEID=$(buildah from ubi8/nodejs-10)

# any build steps above this line run as root
# after this build steps run as $USERID
buildah config --user $USERID:$USERID $IMAGEID

buildah copy $IMAGEID . /opt/app-root/src

# Any other prep steps go here

buildah config --cmd 'npm start' $IMAGEID

buildah commit $IMAGEID $MYIMAGE

```

现在，使`app-image-buils.sh`可执行，然后构建映像:

```
$ chmod +x app-image-build.sh
$ sudo ./app-image-build.sh
```

现在，您可以运行并测试新映像:

```
$ sudo podman run --rm -it --net host myorg/mynodeapp
```

注意，`run`命令不再需要卷挂载，因为代码现在是容器的一部分。

当你准备好了，你可以把你的应用程序推送到一个容器注册中心，比如 Red Hat 的 [Quay.io](https://quay.io/) ，把它分发给其他人。

## 使用 systemd 管理 Node.js 应用程序

您可以使用`systemd`管理您的新 Node.js 应用程序，这样它将在系统启动时启动。以 root 身份创建`systemd`单元文件`/etc/systemd/system/my-node-app.service`，内容如下:

```
[Unit]
Description=My Node App 
After=node-redis-db.service

[Service]
Type=simple
TimeoutStartSec=30s
ExecStartPre=-/usr/bin/podman rm "mynodeapp"

ExecStart=/usr/bin/podman run --name mynodeapp --net host myorg/mynodeapp

ExecReload=-/usr/bin/podman stop "myorg/mynodeapp"
ExecReload=-/usr/bin/podman rm "myorg/mynodeapp"
ExecStop=-/usr/bin/podman stop "myorg/mynodeapp"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target

```

告诉`systemd`重新加载，然后启动应用程序:

```
$ sudo systemctl daemon-reload
$ sudo systemctl start my-node-app
$ systemctl status my-node-app
```

现在，您已经让 Node.js 应用程序和 Redis 在容器中运行，并由`systemd`管理。

## 后续步骤

到目前为止，您应该已经看到在容器中运行所需的软件组件是非常容易的，这样您就可以专注于开发了。感觉应该和不用容器开发没太大区别。希望你能看到如何在这些指导的基础上开发你自己的应用程序。

你应该看看在[红帽容器目录](https://access.redhat.com/containers/#/search/ubi8)中还有哪些 UBI 8 图片可供你使用。如果语言、运行时或服务器不能作为 UBI 映像使用，您可以从`ubi8`基础映像开始构建自己的映像。然后，您可以使用 other 文件中的`yum`命令或`buildah run`添加应用程序流和其他所需的 rpm。

本文中的设置有许多缺点，因为它原本是一个快速且易于理解的演示。有许多方法可以改进设置。例如，带有打包 app 的 Node.js 容器被配置为与`--net host`共享主机的网络，这使得 Node.js 进程通过 localhost 连接 Redis 变得很简单。虽然这种选择对于开发来说既快速又容易，但是您没有容器所提供的网络隔离。

改进网络配置的一个方法是使用 Podman 的 pod 功能，将 web 和数据库容器放在同一个 pod 中，共享名称空间。参见 Brent Baude 的文章 [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)。

## 红帽通用基本图像

我在这篇文章中多次提到 UBI，但没有定义这个术语。UBI 是 Red Hat 的通用基础图像，您可以将其用作容器图像的基础。来自 Mike Guerette 的文章， [Red Hat Universal Base Image:如何在 3 分钟或更短时间内完成:](https://developers.redhat.com/blog/2019/07/29/red-hat-universal-base-image-how-it-works-in-3-minutes-or-less/)

> “Red Hat Universal Base Images (UBI)是符合 OCI 标准的基于容器的操作系统映像，带有补充的运行时语言和可自由再分发的软件包。像以前的 RHEL 基础映像一样，它们是从 Red Hat Enterprise Linux 的一部分构建的。UBI 映像可以从 Red Hat 容器目录中获得，并且可以在任何地方构建和部署。
> 
> “而且，你不需要成为一个红帽客户来使用或重新分发它们。真的。”

随着 Red Hat Enterprise Linux 8 在 5 月份的发布，Red Hat [宣布](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)所有 RHEL 8 基本映像将在新的[通用基本映像最终用户许可协议(EULA)](https://www.redhat.com/licenses/EULA_Red_Hat_Universal_Base_Image_English_20190422.pdf) 下可用。这意味着您可以构建和重新分发使用 Red Hat 的 UBI 映像作为基础的容器映像，而不必切换到基于其他发行版的映像，如 Alpine。换句话说，在构建容器时，您不必从使用`yum`切换到使用`apt-get`。

Red Hat Enterprise Linux 8 有三个基本映像。标准的叫做`ubi`，或者更准确的说是`ubi8/ubi`。这是上面使用的图像，也是您可能最常用的图像。另外两个是最小容器；当图像大小是高优先级时，他们几乎没有支持软件，并且多服务图像允许您在由`systemd`管理的容器内运行多个进程。

**注意:**在`ubi7`下也有适用于 Red Hat Enterprise Linux 7 的 UBI 映像，如果你想构建和分发运行在 RHEL 7 映像上的容器。对于本文，我们只使用`ubi8`图像。

如果您刚刚开始使用容器，现在可能不需要深入 UBI 细节。只需使用`ubi8`映像来构建基于 Red Hat Enterprise Linux 8 的容器。但是，当您开始分发容器映像或有关于支持的问题时，您会希望了解 UBI 的详细信息。有关更多信息，请参阅本文末尾的参考资料。

## 更多信息

相关文章:

*   [在 RHEL 7 上的容器中运行红帽企业 Linux 8](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/)(涵盖了在容器中运行的 PHP 7.2、MariaDB、WordPress)
*   [在 Red Hat Enterprise Linux 上的容器中使用 Django 2 和 Python 3 进行开发](https://developers.redhat.com/blog/?p=624177)
*   [在 Red Hat Enterprise Linux 上用 Flask 和 Python 3 在一个容器中开发](https://developers.redhat.com/blog/?p=624247)

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

*Last updated: January 6, 2022*