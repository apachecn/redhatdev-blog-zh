# 使用 Python 和 Red Hat 容器开发工具包(CDK)构建您的第一个应用程序

> 原文：<https://developers.redhat.com/articles/using-python-with-red-hat-container-development-kit-cdk>

使用红帽容器开发 CDK (CDK) 2 开始在 docker 格式的容器中构建 Python 应用程序

#### 简介和先决条件

在本教程中，您将学习如何在 Red Hat Enterprise Linux 上使用 Red Hat Container Development Kit(CDK)2 开始在 docker 格式的容器中构建 Python 3 应用程序。你需要安装 CDK 2，并且为你的系统下载红帽企业版 Linux 流浪者盒子。更多信息参见 [CDK 2 安装指南](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/container-development-kit-installation-guide/)。

如果您在任一点遇到困难，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 1.启动流浪者之箱

*5 分钟*

本教程中的步骤运行在 Red Hat Enterprise Linux (RHEL)的流浪者之家。流浪者盒子包括 docker、OpenShift Enterprise 和 kubernetes..使用`vagrant ssh`登录到框中后，您将输入本教程中的命令。

打开*终端*或*命令*窗口，输入本教程中的命令。在 Windows 上，建议使用 *Cygwin 终端*而不是*cmd.exe*窗口。

如果您尚未安装 CDK，请遵循 [CDK 安装指南](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/container-development-kit-installation-guide/)。

### 启动流浪者之箱

要启动流浪者之家:

1.  切换到解压 CDK zip 文件的目录。

2.  转到子目录`components/rhel/rhel-ose`。或者，将该目录中的`Vagrantfile`复制到您选择的工作目录中。

3.  通过输入`vagrant up`启动盒子。注意:在没有指定盒名或`Vagrantfile`路径的情况下，输入漫游命令时`Vagrantfile`需要在当前目录下。

4.  启动时会提示你向红帽订阅管理注册流浪者盒子。这是允许机顶盒通过将软件附加到您的 Red Hat 订阅来从 Red Hat 下载软件所必需的。您需要输入您的 Red Hat 用户名和密码。

    流浪者注册插件会在启动时自动将盒子附加到你的红帽订阅上，并在使用`vagrant halt`命令关闭盒子时释放它。

启动流浪盒时，会显示一些日志信息。其中大部分是信息性的，但是如果框启动失败，您应该检查输出。以下是典型`vagrant up`的输出:

```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 => 2222 (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default: Warning: Connection timeout. Retrying...
    default: Warning: Connection timeout. Retrying...
    default: Warning: Remote connection disconnect. Retrying...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: No guest additions were detected on the base box for this VM! Guest
    default: additions are required for forwarded ports, shared folders, host only
    default: networking, and more. If SSH fails on this machine, please install
    default: the guest additions and repackage the box to continue.
    default:
    default: This is not an error message; everything may continue to work properly,
    default: in which case you may ignore this message.
==> default: Configuring and enabling network interfaces...
==> default: Registering box with vagrant-registration...
    default: Would you like to register the system now (default: yes)? [y]y
    default: username: _your username_
    default: password: _your password_
==> default: Rsyncing folder: /cygdrive/c/cdk2/components/rhel/rhel-ose/ => /vagrant
==> default: Running provisioner: shell...
    default: Running: inline script
==> default: Created symlink from /etc/systemd/system/multi-user.target.wants/openshift.service to /etc/systemd/system/openshift.service.
```

你现在应该可以使用`vagrant ssh`登录到流浪者盒子了。

## 2.运行您的第一个容器

*5 分钟*

这一步将使用来自 Red Hat Atomic Registry 的容器映像下载并安装 Python 3，该注册表是容器映像的存储库。安装 Python 3 容器将使 Python 3 可供系统上的其他容器使用。因为容器在隔离的环境中运行，所以安装不会改变您的主机系统。您将使用`docker`命令来交互和查看容器的内容。

本节中显示的命令可用于下载和安装其他容器映像，如您构建的应用程序容器。容器可以指定它们需要安装其他容器，这可以自动发生。例如，您可以在用于描述和构建容器的`Dockerfile`中指定您的应用程序需要 Python 3。然后，当有人安装您的容器时，他们的系统会自动直接从 Red Hat Atomic Registry 下载所需的 Python 3 容器。

Python 3 容器映像是 Red Hat Software Collections 的一部分，它为 Red Hat Enterprise Linux 提供了动态语言、开源数据库和 web 开发工具的最新稳定版本。许多 Red Hat Enterprise Linux (RHEL)订阅都包含对 Red Hat Software Collections(RHS cl)的访问。有关哪些订阅包含 RHSCL 的更多信息，请参见[如何使用 Red Hat 软件集合(RHSCL)或 Red Hat 开发工具集(DTS)](https://access.redhat.com/solutions/472793) 。

在 Red Hat Enterprise Linux 上的 travel box 中运行以下所有命令。如果你还没有登录到流浪者盒子，打开一个*终端*或者*命令*窗口，转到目录`cdk/components/rhel-ose/Vagrantfile`。使用`vagrant ssh`登录

要下载并安装 Python 3 容器映像，请使用以下命令:

`$ docker pull registry.access.redhat.com/rhscl/python-34-rhel7`

`docker images`命令列出了系统中的容器图像:

`$ docker images`

该列表将包括您已经下载的容器和您系统上以前安装的任何容器。CDK 流浪盒包括作为容器映像分发的软件组件。

现在启动一个`bash` shell，查看使用 Python 3 容器映像的容器内部。shell 提示符发生变化，这表明您正在容器内部的 shell 中键入内容。一个`ps -ef`显示容器内唯一运行的东西是`bash`和`ps`。键入`exit`离开容器的 bash shell。

```
$ docker run -it rhscl/python-34-rhel7 /bin/bash
bash-4.2$ which python3
/opt/rh/rh-python34/root/usr/bin/python3
bash-4.2$ python3 --version
Python 3.4.2
bash-4.2$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
default      1     0  0 17:58 ?        00:00:00 /bin/bash
default     10     1  0 17:58 ?        00:00:00 ps -ef
bash-4.2$ exit
```

前面的`docker run`命令创建了一个容器来运行您的命令，保持任何状态，并将其与系统的其余部分隔离。您可以使用`docker ps`查看正在运行的容器列表。要查看所有已经创建的容器，包括那些已经退出的，使用`docker ps -a`。取决于你使用的是哪一个浮动文件，可能有许多其他的容器在运行，比如用于创建一个 OpenShift 环境的容器。

您可以使用`docker start`重新启动上面创建的容器。容器是通过名称来引用的。如果你不提供名字，Docker 会自动生成一个名字。一旦容器重新启动，`docker attach`将允许您与运行在其中的 shell 交互。请参见以下示例:

```
$ docker ps -a
CONTAINER ID        IMAGE                        COMMAND                  CREATED              STATUS                          PORTS               NAMES
d949277c36e9        rhscl/python-34-rhel7        "container-entrypoint"   About a minute ago   Exited (0) About a minute ago                       determined_mayer

$ docker start determined_mayer
determined_mayer
$ docker attach determined_mayer
```

此时，您已经连接到容器内部正在运行的 shell。当你连接时，你不会看到命令提示符，所以按下回车键让它打印另一个。

```
bash-4.2$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
default      1     0  0 18:01 ?        00:00:00 /bin/bash
default      9     1  0 18:01 ?        00:00:00 ps -ef
bash-4.2$ exit
```

由于容器中唯一的进程`bash`被告知`exit`，容器将不再运行。这可以用`docker ps -a`来验证。不再需要的容器可以用`docker rm *<container-name>*`清理。

`$ docker rm determined_mayer`

要查看 Red Hat 容器注册表中还有哪些容器图像，请使用以下一个或多个搜索:

```
$ docker search registry.access.redhat.com/rhscl
$ docker search registry.access.redhat.com/openshift3
$ docker search registry.access.redhat.com/rhel
$ docker search registry.access.redhat.com/jboss
```

如果您需要帮助，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 3.在容器中构建 Hello World

*5 分钟*

在这一步中，您将创建一个使用 Python 3 作为 web 服务器的小型 Hello World 容器。一旦创建，容器就可以在安装了`docker`的其他系统上运行。您将需要使用您喜欢的编辑器在一个空目录中创建几个文件，包括一个描述如何构建容器映像的`Dockerfile`。

注意:你可以在你的主机系统上编辑文件，这些文件可以通过`vagrant rsync`同步到你的流浪箱。更多信息参见 *CDK 安装指南*中的*流浪同步文件夹*。

首先创建一个空目录，然后创建一个名为`index.html`的文件，内容如下:

index.html

```
<html>Hello, Red Hat Developers World from Python 3!</html>
```

现在，在同一个目录中，创建一个名为`Dockerfile`的文件，包含以下内容，但是将`MAINTAINER`行改为包含您的姓名和电子邮件地址:

Dockerfile

```
FROM rhscl/python-34-rhel7:latest

MAINTAINER Your Name "your-email@example.com"

EXPOSE 8000

COPY . /opt/app-root/src

CMD /bin/bash -c 'python3 -u web.py'
```

在与`Dockerfile`相同的目录下创建文件`web.py`

web.py

```
#
# A very simple Python HTTP server
#

import http.server
import socketserver

PORT = 8000

Handler = http.server.SimpleHTTPRequestHandler

httpd = socketserver.TCPServer(("", PORT), Handler)

print("serving at port", PORT)
httpd.serve_forever()
```

现在使用`docker build`构建容器映像。

`$ docker build -t *myname*/pythonweb .`

您可以看到使用以下命令创建的容器映像:

`$ docker images`

现在使用`docker run`运行容器。Python 3 http.server 模块将创建一个小型 web 服务器，它监听容器内部的端口 8000。`run`命令将主机上的端口 8000 映射到容器内的端口 8000。

`$ docker run -d -p 8000:8000 --name helloweb *myname*/pythonweb`

run 命令返回容器的唯一 ID，您可以忽略它。要检查容器是否正在运行，请使用`docker ps`。输出应该显示一个名为`helloweb`的容器，它正在运行您创建的`*myname*/pythonweb`容器映像。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
c7885aa23773        myname/pythonweb    "container-entrypoint"   6 seconds ago       Up 4 seconds        0.0.0.0:8000->8000/tcp, 8080/tcp   helloweb
```

使用`curl`访问 Python web 服务器:

```
# curl http://localhost:8000/
<html>Hello, Red Hat Developers World from Python 3!</html>
```

注意:您还应该能够从主机上的浏览器访问运行在容器中的 Python web 服务器。`rhel-ose/Vagrantfile`将无线箱的 IP 地址设置为 10.1.2.2。在您的主机系统上使用的 url 是`[http://10.1.2.2:8000/](http://10.1.2.2:8000/)`。

要查看运行容器中的日志，请使用`docker logs *<container-name>*`:

`$ docker logs helloweb`

完成后，停止正在运行的容器:

`$ docker stop helloweb`

`helloweb`容器将被保留，直到您用`docker rm`将其移除。你可以用`docker start helloweb`重启容器。注意:如果同名的容器已经存在，后续的`docker run`将产生一个错误。

您可以使用`docker inspect`查看有关容器的信息:

`$ docker inspect *myname*/pythonweb`

输出是一个易于阅读的 JSON 结构。 *Config* 部分包含容器运行时环境的细节，比如环境变量和默认命令。注意，容器配置中的许多信息都是从父容器继承的，在本例中是 Python 3 运行时容器。

最后，当您创建的应用程序容器映像准备就绪时，您可以通过将它们推送到公共或私有容器注册中心来分发它们。然后，您的容器将可以使用`docker pull`安装到其他系统上。

**想知道更多关于容器开发工具包的事情吗？**

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**关注红帽开发者博客**【https://developers.redhat.com/blog 

## 故障排除和常见问题

1.  我如何知道是否有一个容器映像有一个新版本的 PHP？

    我怎样才能看到其他可用的容器图像？

    我找不到本教程中提到的容器，如何判断名称是否更改？

    要查看 Red Hat 容器注册表中还有哪些容器，请使用以下一个或多个搜索:

    ```
    $ docker search registry.access.redhat.com/rhscl
    $ docker search registry.access.redhat.com/openshift3
    $ docker search registry.access.redhat.com/rhel
    $ docker search registry.access.redhat.com/jboss
    ```

2.  在哪里可以了解更多关于使用 Linux 容器交付应用程序的信息？

    如果你还没有加入[红帽开发者计划](https://developers.redhat.com/)，请在 developers.redhat.com[注册。会员是免费的。](https://developers.redhat.com/) [容器开发的推荐实践](https://access.redhat.com/articles/1483053)和许多其他容器文章可从 [Red Hat 客户门户](https://access.redhat.com/)获得。

    如果您是红帽技术合作伙伴，请访问[容器专区](https://access.redhat.com/articles/1483053)或[红帽技术合作伙伴连接](http://connect.redhat.com/)网站。

*Last updated: November 19, 2020*