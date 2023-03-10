# 在 Red Hat Enterprise Linux 的容器中使用 Django 2 和 Python 3 进行开发

> 原文：<https://developers.redhat.com/blog/2019/09/11/develop-with-django-2-and-python-3-in-a-container-with-red-hat-enterprise-linux>

在我之前的文章[在 RHEL 7](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/) 上的容器中运行红帽企业 Linux 8 中，我展示了如何开始使用红帽企业 Linux 8 可用的最新版本的语言、数据库和 web 服务器进行开发，即使您仍然运行 RHEL 7。在本文中，我将在此基础上展示如何使用当前的 RHEL 8 应用程序流版本 Python 3 和 PostgreSQL 10 开始使用 Django 2。

在我看来，在容器中使用 Red Hat Enterprise Linux 8 应用程序流比在 RHEL 7 上使用软件集合更好。虽然您需要熟悉容器，但是所有的软件都安装在您期望的位置。不需要使用`scl`命令来管理选择的软件版本。相反，每个容器都有一个独立的用户空间。你不必担心版本冲突。

在本文中，我将向您展示如何使用 Buildah 创建一个 Red Hat Enterprise Linux 8 Django 容器，并使用 Podman 运行它。代码存储在本地机器上，并在运行时映射到容器中。您可以像编辑任何其他应用程序一样在本地机器上编辑代码。因为它是通过卷挂载来映射的，所以您对代码所做的更改可以立即从容器中看到，这对于不需要编译的动态语言来说很方便。

虽然这种方法不是您想要的生产方式，但您得到的开发内循环本质上与没有容器的本地开发相同。本文还展示了如何使用 Buildah 为您完成的应用程序构建一个映像，以便用于生产。此外，您将在由`systemd`管理的容器中设置 RHEL 8 PostgreSQL 应用程序流。您将能够使用`systemctl`来启动和停止容器，就像您对非容器安装一样。

## 准备 Red Hat Enterprise Linux 7

在我们在 RHEL 7 系统上创建 Red Hat Enterprise Linux 8 容器之前，我们需要确保安装了必要的软件，然后选择一个示例 Django 应用程序或创建我们自己的应用程序。让我们来看一下这个准备阶段。

### 在 RHEL 7 号上安装波德曼和 Buildah

首先我们需要安装 Podman，它在红帽企业版 Linux 7 `extras` repo 中。`extras`回购默认不启用。建议开发者也启用`rhscl` ( [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview))、`devtools`、`optional`回购。要同时启用所有这些功能:

```
$ sudo subscription-manager repos --enable rhel-7-server-extras-rpms \
    --enable rhel-7-server-optional-rpms \
    --enable rhel-server-rhscl-7-rpms \
    --enable rhel-7-server-devtools-rpms
```

现在安装 Podman 和 Buildah:

```
$ sudo yum install podman buildah
```

**提示:**如果您的系统上没有安装`sudo`，请参见[如何在 Red Hat Enterprise Linux](https://developers.redhat.com/blog/2018/08/15/how-to-enable-sudo-on-rhel/) 上启用 sudo。

稍后，我们将运行带有`systemd`的容器。如果在您的系统上启用了 SELinux(默认情况下)，您必须打开`container_manage_cgroup`布尔值来运行带有`systemd`的容器:

```
$ sudo setsebool -P container_manage_cgroup on
```

有关更多信息，请参见运行 systemd 解决方案的[容器。](https://access.redhat.com/solutions/3387631)

**注意:**您加入 Red Hat Developer 时创建的 Red Hat ID 允许您访问 [Red Hat 客户门户网站上的内容](https://access.redhat.com)。

### 设置 Django 示例应用程序

我们需要一些 Django 代码来运行。我们将使用[编写你的第一个 Django 应用](https://docs.djangoproject.com/en/2.2/intro/tutorial01/)教程中的投票应用。我们将从 GitHub 中提取代码，而不是手动重新创建所有文件。Django 项目并没有让示例应用程序易于下载，但一些用户已经创建了包含它的 repos。我选择 [`monim67/django-polls`](https://github.com/monim67/django-polls) 是因为教程每一章的代码都被标记在 repo 中。

运行以下命令创建源目录:

```
$ sudo mkdir /opt/src
$ sudo chown $USER:$USER /opt/src
$ cd /opt/src
$ git clone https://github.com/monim67/django-polls.git
$ cd polls-app
$ git tag  # see what tags are available
$ git checkout d2.1t7 # optionally checkout the code for the last chapter
```

我们现在在`/opt/src/django-polls`有一个示例 Django 应用程序。

## 创建您的 Red Hat Enterprise Linux 8 容器映像

现在我们的 Red Hat Enterprise Linux 7 准备工作已经完成，我们可以创建自定义的 RHEL 8 容器映像了。使用现有的 Red Hat 通用基础映像(UBI)可以加速这个过程。

### 了解红帽通用基本图像

通用基础图像是来自 Red Hat 的通用基础图像，您可以将其用作容器图像的基础。来自 Mike Guerette 的文章，“ [Red Hat Universal Base Image:如何在 3 分钟或更短时间内工作](https://developers.redhat.com/blog/2019/07/29/red-hat-universal-base-image-how-it-works-in-3-minutes-or-less/):”

> “Red Hat Universal Base Images (UBI)是符合 OCI 标准的基于容器的操作系统映像，带有补充的运行时语言和可自由再分发的软件包。像以前的 RHEL 基础映像一样，它们是从 Red Hat Enterprise Linux 的一部分构建的。UBI 映像可以从 Red Hat 容器目录中获得，并且可以在任何地方构建和部署。
> 
> “而且，你不需要成为一个红帽客户来使用或重新分发它们。真的。”

随着 Red Hat Enterprise Linux 8 在 5 月份的发布，Red Hat [宣布](https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image)所有 RHEL 8 基本映像将在新的[通用基本映像最终用户许可协议(EULA)](https://www.redhat.com/licenses/EULA_Red_Hat_Universal_Base_Image_English_20190422.pdf) 下可用。这一事实意味着您可以构建和重新分发使用 Red Hat 的 UBI 映像作为基础的容器映像，而不必切换到基于其他发行版的映像，如 Alpine。换句话说，在构建容器时，您不必从使用`yum`切换到使用`apt-get`。

RHEL 8 有三个基本映像。标准的叫做`ubi`，或者更准确的说是`ubi8/ubi`。这是上面使用的图像，也是您可能最常用的图像。另外两个是最小容器。当图像大小是高优先级时，他们几乎没有支持软件，并且多服务图像允许您在由`systemd`管理的容器内运行多个进程。

**注意:**如果您想构建和分发在 RHEL 7 映像上运行的容器，在`ubi7`下也有 RHEL 7 的 UBI 映像。对于本文，我们只使用`ubi8`图像。

如果您刚刚开始使用容器，现在可能不需要深入 UBI 细节。仅仅使用`ubi8`图像来建造基于 RHEL 8 的容器。但是，当您开始分发容器映像或对支持有疑问时，您需要了解 UBI 的详细信息。有关更多信息，请参阅本文末尾的参考资料。

### 将 Python 3.6 和 Django 添加到 RHEL 8 容器中(手动)

现在我们需要 Python 3.6 和 Django。我们将设置一个手动安装依赖项的容器，然后运行应用程序，看看它是如何完成的。

我们用红帽企业 Linux 8 [通用基础镜像(UBI)](https://developers.redhat.com/products/rhel/ubi/) 吧。但是首先要登录新的 Red Hat 容器注册中心，它支持认证， [registry.redhat.io](https://registry.redhat.io) 。如果您在尝试提取映像时没有登录，您将收到一条详细的错误消息，其中包含以下消息:

```
...unable to retrieve auth token: invalid username/password
```

使用您的 Red Hat 开发人员用户名和密码登录注册表:

```
$ sudo podman login registry.redhat.io
```

**注:**波德曼被设计成可以在没有 root 的情况下运行。然而，RHEL 7.6 不支持该特性。更多信息，请参见 Scott McCarty 的[在 RHEL 7.6](https://www.redhat.com/en/blog/preview-running-containers-without-root-rhel-76) 中运行无根容器的预览。

现在，运行容器并使源目录(`/opt/src`)在容器内可用，然后暴露端口 8000，这样您就可以使用主机系统上的浏览器连接到 Django 应用程序:

```
$ sudo podman run -it -v /opt/src:/opt/src:Z -p 8000:8000 registry.redhat.io/ubi8/ubi /bin/bash
```

在容器内部，查看 Red Hat Enterprise Linux 8 提供了哪些应用流:

```
# yum module list
```

**注意:**您可能会注意到一组额外的应用程序流，标记为“通用基础映像”

接下来，安装 Python 3.6:

```
# yum -y module install python36
```

Python 3.6 现在已经安装在我们的容器中，并作为`python3`而不是`python`位于我们的路径中。如果你想知道为什么，可以看看 Petr Viktorin 的文章， [Python 在 RHEL 8](https://developers.redhat.com/blog/2018/11/14/python-in-rhel-8-3) 。

接下来使用`pip`来安装 Django:

```
# pip3 install django
```

您将得到一个关于以 root 用户身份运行`pip`的警告。在真实系统上以 root 身份运行`pip`通常不是一个好主意。然而，我们运行在一个隔离的一次性专用容器中，所以我们可以对`/usr`中的文件做任何我们想做的事情。

让我们检查一下`django-admin`命令行界面(CLI)的安装位置:

```
# which django-admin
```

`pip`命令将 CLI 安装到`/usr/local/bin`中。现在，让我们在容器中运行示例应用程序:

```
# cd /opt/src/django-polls
# python3 manage.py runserver 0:8000
```

**注意:**当您运行应用程序时，您可能会看到一条关于*未应用迁移*的错误消息。如果您使用上述 repo，可以忽略此消息，它包括一个预填充的 SQLite 数据库。如果您使用了不同的源或者需要创建数据库，请参见下面的“在容器中运行 DB 迁移和 Django admin”。

使用主机系统上的浏览器，转到`http://localhost:8000/admin/`。来自 GitHub repo 的 SQLite 数据库的用户名和密码是`admin`和`admin`。登录后，您应该会看到如下屏幕:

![](img/5409c29f6c6916f3c349433b41007bb2.png)现在，您已经有了一个手工配置的容器，它将使用 Red Hat Enterprise Linux 8 的 Python 3.6 应用程序流在您的 RHEL 7 系统上运行 Django。你可以像对待宠物一样对待这个容器，当你想再次运行它的时候使用`podman restart -l` 和`podman attach -l`，只要你不删除它。我们没有命名容器，但是`-l`方便地选择了最后一个运行的容器。尽管知道这些信息对于测试来说很方便，但是它很少是可重复的。

这个容器的另一个问题是，它以 root 用户身份运行，以便能够安装软件。由`/opt/src`中的容器创建的任何文件都将归 root 所有。

### 用 buildhr 创建 django 容器映像

为了使事情变得更简单，我们将创建一个安装了 Django 的容器映像，并在创建容器时启动 Django 应用程序。容器中没有应用程序的副本，我们仍将它从主机系统映射到容器中。代码将存储在您的本地机器上，您可以像编辑任何其他应用程序源一样编辑它。因为它是通过卷挂载来映射的，所以您对代码所做的更改将在容器内部立即可见。

使用 Buildah 创建图像时，可以使用 Dockerfiles 或 Buildah 命令行。对于本文，我们将使用 Dockerfile 方法，因为其他教程经常使用这种方法。

因为我们使用的是在您的主机系统和容器之间共享的文件，所以我们将使用与您的常规帐户相同的数字用户 ID (UID)来运行容器。当在容器中时，在源目录中创建的任何文件都将归您的主机系统用户 ID 所有。用`id`命令找出你的 UID:

```
$ id
```

记下该行最开始的`UID=`和`GID=`后的数字。在我的系统上，我的 UID 和 GID 都是 1000。在 other 文件和下面的其他例子中，修改`USER`行以匹配您的 UID:GID。

在`/opt/src/django-polls`中，用以下内容创建`Dockerfile`:

```
FROM registry.redhat.io/ubi8/python-36

RUN pip3 install django gunicorn psycopg2

# This is primarily a reminder that we need access to port 8000
EXPOSE 8000

# Change this to UID that matches your username on the host
# Note: RUN commands before this line will execute as root in the container
# RUN commands after will execute under this non-privileged UID
USER 1000:1000

# Default cmd when container is started
# Default directory was already set by Python container to /opt/app-root/src
# Get Django to listen on all interfaces so we can connect from outside the container
CMD python3 manage.py runserver 0:8000

```

文档上的一些注释。我没有安装 Python 3.6，而是使用了 Red Hat 的 UBI 映像，该映像已经在 UBI 8 映像上安装了 Python 3.6。在构建过程中，`pip`将作为根用户在容器中运行，因为它位于变更为非特权用户的`USER`行之上。

接下来，构建 Django 容器:

```
$ sudo buildah bud -t myorg/mydjangoapp .
```

(别忘了结尾的`.` )

现在，运行 Django 容器，这将启动 Polls 应用程序:

```
$ sudo podman run --rm -it -p 8000:8000 -v /opt/src/django-polls:/opt/app-root/src:Z myorg/mydjangoapp
```

Django polls 应用程序现在应该正在运行，您可以使用主机系统上的浏览器并转到`http://localhost:8000/admin`进行验证。

您现在可以像编辑任何常规源代码一样编辑`/opt/src/django-polls`中的代码。当您需要重新启动时，CTRL+C 容器。注意`run`命令中的`--rm`，它会在容器退出时自动移除容器。要再次启动容器，再次使用上面的`podman run`命令，这将创建一个新的容器。

## 设置您的数据库

现在要建立一个持久的、全功能的数据库，当你关闭容器时它不会消失，然后把它连接到 Django。

### 在容器中运行数据库迁移和 Django 管理

因为运行 Django 的环境存在于容器中，所以您需要在其中运行任何 Django 管理命令。您可以运行一个命令，也可以在容器内部启动一个 shell。要应用任何数据库迁移:

```
$ sudo podman run --rm -it -p 8000:8000 -v /opt/src/django-polls:/opt/app-root/src:Z myorg/mydjangoapp python3 manage.py migrate
```

或者，要在容器内交互工作，请启动一个 shell:

```
$ sudo podman run --rm -it -p 8000:8000 -v /opt/src/django-polls:/opt/app-root/src:Z myorg/mydjangoapp /bin/bash
```

**注意:**shell 提示符被 Python 基映像设置为`(app-root)`。

如果您想初始化一个新的数据库:

```
(app-root) python3 rm db.sqlite3
(app-root) python3 manage.py migrate
(app-root) python3 manage.py createsuperuser
```

您可以留在 shell 中运行应用程序:

```
(app-root) python3 manage.py runserver 0:8000
```

### 确保数据库持久性

默认情况下，Django polls 应用程序使用文件`db.sqlite3`中的 SQLite 数据库，该文件与源代码一起位于`django-polls`目录中。因为我们已经将容器设置为从主机映射到一个目录中，所以数据库将随着我们的源代码在容器运行之间持久化。

因为容器是短暂的，如果数据库存储在`/opt/app-root/src`目录之外，它不会在容器的运行之间持续。您可以通过对`podman run`命令应用另一个卷挂载`-v`来解决这个问题。

除了使用 SQLite，您可能还想使用独立于 Django 应用程序容器运行的完整数据库服务器。

### 在容器中运行 PostgreSQL 10

在本节中，我们将让 Red Hat Enterprise Linux 8 PostgreSQL 10 应用程序流在由主机系统上的`systemd`管理的容器中运行。搜索[红帽容器目录](https://access.redhat.com/containers/)，我们可以寻找 PostgreSQL 图像。在撰写本文时，目录中还没有基于 UBI 8 的 PostgreSQL 映像，但是有一个基于 RHEL 8 的[映像。为了便于检查，我们将在运行前提取图像:](https://access.redhat.com/containers/#/registry.access.redhat.com/rhel8/postgresql-10)

```
$ sudo podman pull registry.redhat.io/rhel8/postgresql-10
```

由于容器被设计成短暂的，我们需要为数据库建立永久存储。我们将在主机系统上建立一个目录，并将其映射到容器中。首先，检查图像，找出目录所需的用户 ID。或者，我们也可以从它的 [Red Hat Container 目录页面](https://access.redhat.com/containers/#/registry.access.redhat.com/rhel8/postgresql-10)获得关于这个图像的信息:

```
$ sudo podman inspect postgresql-10 | grep -A 1 User
```

现在我们已经获得了容器将运行的用户 ID，在主机上创建一个目录，赋予该用户 ID 所有权，并为 SELinux 设置上下文:

```
$ sudo mkdir -p /opt/dbdata/django-polls-db
$ sudo chown 26:26 /opt/dbdata/django-polls-db
$ sudo setfacl -m u:26:-wx /opt/dbdata/django-polls-db
$ sudo semanage fcontext -a -t container_file_t /opt/dbdata/django-polls-db
$ sudo restorecon -v /opt/dbdata/django-polls-db
```

让我们手工测试一下 PostgreSQL:

```
$ sudo podman run -it --name django-polls-pgsql -e POSTGRESQL_USER=polls -e POSTGRESQL_PASSWORD=mysecret -e POSTGRESQL_DATABASE=django-polls -p 5432:5432 -v /opt/dbdata/django-polls-db:/var/lib/pgsql/data:Z registry.redhat.io/rhel8/postgresql-10
```

您应该会看到如下所示的输出:

```
Success. You can now start the database server using:

pg_ctl -D /var/lib/pgsql/data/userdata -l logfile start

waiting for server to start....
2019-08-18 21:10:08.545 UTC [31] LOG: listening on Unix socket "/tmp/.s.PGSQL.5432"
2019-08-18 21:10:08.554 UTC [31] LOG: redirecting log output to logging collector process
2019-08-18 21:10:08.554 UTC [31] HINT: Future log output will appear in directory "log".
done
server started
/var/run/postgresql:5432 - accepting connections
=> sourcing /usr/share/container-scripts/postgresql/start/set_passwords.sh ...
ALTER ROLE
waiting for server to shut down....
server stopped
Starting server...
2019-08-18 21:10:09.136 UTC [1] LOG: listening on IPv4 address "0.0.0.0", port 5432
2019-08-18 21:10:09.136 UTC [1] LOG: listening on IPv6 address "::", port 5432
2019-08-18 21:10:09.139 UTC [1] LOG: listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2019-08-18 21:10:09.142 UTC [1] LOG: listening on Unix socket "/tmp/.s.PGSQL.5432"
2019-08-18 21:10:09.160 UTC [1] LOG: redirecting log output to logging collector process
2019-08-18 21:10:09.160 UTC [1] HINT: Future log output will appear in directory "log".
```

PostgreSQL 已正确启动，因此您可以 CTRL+C。然后通过移除容器进行清理:

```
$ sudo podman rm django-polls-pgsql
```

接下来，创建一个`systemd`单元文件来管理 PostgreSQL。作为 root 用户，使用编辑器或`cat >`创建`/etc/systemd/system/django-polls-pgsql.service`，内容如下:

```
[Unit]
Description=Django Polls PostgreSQL Database
After=network.target

[Service]
Type=simple
TimeoutStartSec=5m
ExecStartPre=-/usr/bin/podman rm "django-polls-pgsql"

ExecStart=/usr/bin/podman run --name django-polls-pgsql -e POSTGRESQL_USER=polls -e POSTGRESQL_PASSWORD=mysecret -e POSTGRESQL_DATABASE=django-polls -p 5432:5432 -v /opt/dbdata/django-polls-db:/var/lib/pgsql/data registry.redhat.io/rhel8/postgresql-10

ExecReload=-/usr/bin/podman stop "django-polls-pgsql"
ExecReload=-/usr/bin/podman rm "django-polls-pgsql"
ExecStop=-/usr/bin/podman stop "django-polls-pgsql"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target

```

接下来，告诉`systemd`重新加载，启动 PostgreSQL 服务，然后检查输出:

```
$ sudo systemctl daemon-reload
$ sudo systemctl start django-polls-pgsql
$ sudo systemctl status django-polls-pgsql
```

您可以使用以下命令检查容器日志:

```
$ sudo podman logs django-polls-pgsql
```

PostgreSQL 端口 5432 对主机系统公开。因此，如果您安装了客户端，您应该能够连接到数据库。

关于`systemd`单元文件中的`podman run`命令，有几点需要注意。不要像从命令行那样使用`-d`选项来脱离正在运行的容器。因为`systemd`正在管理这个过程，所以`podman run`不应该退出，直到容器死亡。如果你有一个`-d`，`systemd`会认为容器有故障，会重启它。

不使用退出时自动移除容器的`--rm`到`podman run`选项。相反，`systemd`被配置为在启动容器之前运行一个`podman rm`命令。这使您有机会在停止的容器退出后检查其内部文件的状态。

### 配置 Django 使用 PostgreSQL

您需要更改包含 Polls 应用程序的 Django 站点的设置，以使用 PostgreSQL 数据库而不是 SQLite。编辑`/opt/src/django-polls/mysite/settings.py`并将`DATABASES`部分改为:

```
DATABASES = {
  'default': {
    # 'ENGINE': 'django.db.backends.sqlite3',
    # 'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    'ENGINE': 'django.db.backends.postgresql_psycopg2',
    'NAME': 'django-polls',
    'USER': 'polls',
    'PASSWORD': 'mysecret',
    'HOST': 'localhost',
  }
}

```

## 使用 Buildah 创建 Django 应用程序的图像

开发完应用程序后，您可以使用 Buildah 为 Django 应用程序创建一个可分发的容器映像。我们将使用 Buildah 命令行，而不是 Dockerfile 文件。这种方法对于复杂的构建和自动化来说更加灵活。您可以使用 shell 脚本或任何用于构建环境的工具:

```
#!/bin/sh
# Build our Django app and all the dependencies into a container image
# Note: OOTB on RHEL 7.6 this needs to be run as root.
MYIMAGE=myorg/mydjangoapp
USERID=1000

IMAGEID=$(buildah from ubi8/python-36)
buildah run $IMAGEID pip3 install django gunicorn psycopg2

# any build steps above this line run as root
# after this build steps run as $USERID
buildah config --user $USERID:$USERID $IMAGEID

buildah copy $IMAGEID . /opt/app-root/src

# Any other prep steps go here. you could apply migrations, etc.
buildah config --cmd 'python3 manage.py runserver 0:8000' $IMAGEID

buildah commit $IMAGEID $MYIMAGE

```

现在，使`app-image-buils.sh`可执行，然后构建映像:

```
$ chmod +x app-image-build.sh
$ sudo ./app-image-build.sh
```

现在，您可以运行并测试新映像:

```
$ sudo podman run --rm -it --net host myorg/mydjangoapp
```

`run`命令不再需要卷挂载，因为代码现在在容器中，数据由 PostgreSQL 容器的卷管理。

当你准备好了，你可以把你的应用程序推送到一个容器注册中心，比如 Red Hat 的 [Quay.io](https://quay.io/) ，把它分发给其他人。

## 用系统管理你的 django 应用

你可以用`systemd`管理你的新 Django 应用程序，这样它就会在系统启动时启动。以 root 身份创建`systemd`单元文件`/etc/systemd/system/django-polls-app.service`，内容如下:

```
[Unit]
Description=Django Polls App

After=django-polls-pgsql.service

[Service]
Type=simple
TimeoutStartSec=30s
ExecStartPre=-/usr/bin/podman rm "mydjangoapp"

ExecStart=/usr/bin/podman run --name mydjangoapp --net host myorg/mydjangoapp

ExecReload=-/usr/bin/podman stop "myorg/mydjangoapp"
ExecReload=-/usr/bin/podman rm "myorg/mydjangoapp"
ExecStop=-/usr/bin/podman stop "myorg/mydjangoapp"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

告诉`systemd`重新加载，然后启动应用程序。

```
$ sudo systemctl daemon-reload
$ sudo systemctl start django-polls-app
```

## 后续步骤

到目前为止，您应该看到在容器中运行所需的软件组件非常容易，这样您就可以专注于开发。感觉应该和不用容器开发没太大区别。希望你能看到如何在这些指导的基础上开发你自己的应用程序。

您应该在 Red Hat 容器目录中查看还有哪些 UBI 8 图像可供您使用。如果语言、运行时或服务器不能作为 UBI 映像使用，您可以使用 UBI 8 基础映像构建自己的映像。然后，您可以使用 other 文件中的`yum`命令或`buildah run`添加应用程序流和其他所需的 rpm。

本文中的设置有许多缺点，因为它原本是一个快速且易于理解的演示。有许多方法可以改进设置。例如，带有打包应用程序的 Django 容器被配置为与`--net host`共享主机的网络，这使得 Django 进程通过 localhost 连接到数据库变得很简单。虽然这种设置对于开发来说既快速又简单，但是您无法获得容器所提供的网络隔离。

改进网络配置的一个方法是使用 Podman 的 pod 功能，将 web 和数据库容器放在同一个 pod 中，在这个 pod 中它们共享名称空间。参见 Brent Baude 的文章 [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)。

## 更多信息

相关文章:

*   [在 RHEL 7 上的容器中运行红帽企业 Linux 8](https://developers.redhat.com/blog/2019/08/23/run-red-hat-enterprise-linux-8-in-a-container-on-rhel-7/)(涵盖了在容器中运行的 PHP 7.2、MariaDB、WordPress。)
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
*   [红帽开发者的 UBI 页面](https://developers.redhat.com/products/rhel/ubi/)
*   [UBI 常见问题](https://developers.redhat.com/articles/ubi-faq/)

*Last updated: January 13, 2022*