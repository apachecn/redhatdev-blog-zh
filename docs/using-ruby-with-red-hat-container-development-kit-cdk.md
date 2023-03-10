# 使用 Ruby with Red Hat 容器开发工具包(CDK)构建您的第一个应用程序

> 原文：<https://developers.redhat.com/articles/using-ruby-with-red-hat-container-development-kit-cdk>

使用 Red Hat Container Development CDK(CDK)2 开始在 docker 格式的容器中构建 Ruby 应用程序。

#### 简介和先决条件

在本教程中，您将学习如何在 Red Hat Enterprise Linux 上使用 Red Hat Container Development Kit(CDK)2 在 docker 格式的容器中构建 Ruby 应用程序。你需要安装 CDK 2，并且为你的系统下载红帽企业版 Linux 流浪者盒子。更多信息参见 [CDK 2 安装指南](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/container-development-kit-installation-guide/)。

如果您在任一点遇到困难，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 1.启动流浪者之箱

*5 分钟*

本教程中的步骤运行在 Red Hat Enterprise Linux (RHEL)上，流浪者之家是 CDK 的一部分。流浪者盒子安装并配置了 docker、kubernetes 和 OpenShift 来运行容器。使用`vagrant ssh`登录到框中后，您将输入本教程中的命令。

打开*终端*或*命令*窗口，输入本教程中的命令。

如果您尚未安装 CDK，请遵循 [CDK 安装指南](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/container-development-kit-installation-guide/)。步骤概述:

1.  为您的系统安装一个虚拟化提供程序(虚拟机管理程序),例如 VirtualBox、VMware 或 KVM/libvirt。

2.  安装流浪者，从[vagrantup.com](https://www.vagrantup.com/)，或者从你的操作系统的软件包。

3.  从[红帽客户门户](https://access.redhat.com/downloads/content/293/ver=2/rhel---7/2.0.0/x86_64/product-software)下载*红帽容器工具* (cdk-2.0.0.zip)和 *RHEL 7.2 流浪盒*匹配您的虚拟化提供商:VirtualBox、VMare 或 KVM/libvirt。

4.  将`cdk-2.0.0.zip`解压到您选择的目录中。

5.  安装流浪插件`vagrant-registration`和`vagrant-adbinfo`。插件位于 CDK zip 文件的`plugins`目录中。

6.  用命令`vagrant add --name cdkv2 *path to downloaded .box file*`添加 *RHEL 7.2 流浪盒*进行流浪。

### 启动流浪者之箱

要启动流浪者之家:

1.  切换到解压 CDK zip 文件的目录。

2.  转到子目录`components/rhel/rhel-ose`。或者，将该目录中的`Vagrantfile`复制到您选择的工作目录中。

3.  通过输入`vagrant up`启动盒子。注意:在没有指定盒名或`Vagrantfile`路径的情况下，输入漫游命令时`Vagrantfile`需要在当前目录下。

4.  启动时会提示你注册 RHEL 流浪箱。您需要为客户门户网站[access.redhat.com](https://access.redhat.com/)输入您的用户名和密码。必须注册机器才能从 Red Hat 下载软件。注册插件会在启动时将盒子附加到 Red Hat 订阅，并在使用`vagrant halt`命令关闭盒子时释放它。

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

这一步将使用 Red Hat Atomic Registry(一个容器映像库)中的容器映像下载并安装 Ruby。安装 Ruby 容器将使 Ruby 可供系统上的其他容器使用。因为容器在隔离的环境中运行，所以安装不会改变您的主机系统。您将使用`docker`命令来交互和查看容器的内容。

本节中显示的命令可用于下载和安装其他容器映像，如您构建的应用程序容器。容器可以指定它们需要安装其他容器，这可以自动发生。例如，您可以在用于描述和构建您的容器的`Dockerfile`中指定您的应用程序需要 Ruby。然后，当有人安装您的容器时，他们的系统会自动直接从 Red Hat Atomic Registry 下载所需的 Ruby 容器。

Ruby 容器映像是 Red Hat Software Collections 的一部分，它为 Red Hat Enterprise Linux 提供了动态语言、开源数据库和 web 开发工具的最新稳定版本。许多 Red Hat Enterprise Linux (RHEL)订阅都包含对 Red Hat Software Collections(RHS cl)的访问。有关哪些订阅包含 RHSCL 的更多信息，请参见[如何使用 Red Hat 软件集合(RHSCL)或 Red Hat 开发工具集(DTS)](https://access.redhat.com/solutions/472793) 。

在 Red Hat Enterprise Linux 上的 travel box 中运行以下所有命令。如果你还没有登录到流浪者盒子，打开一个*终端*或者*命令*窗口，转到目录`cdk/components/rhel-ose/Vagrantfile`。使用`vagrant ssh`登录

要下载并安装 Ruby 容器映像，请使用以下命令:

`$ docker pull registry.access.redhat.com/rhscl/ruby-22-rhel7`

`docker images`命令列出了系统中的容器图像:

`$ docker images`

该列表将包括您已经下载的容器和您系统上以前安装的任何容器。CDK 流浪盒包括作为容器映像分发的软件组件。

现在启动一个`bash` shell 来查看使用 Ruby 容器映像的容器内部。shell 提示符发生变化，这表明您正在容器内部的 shell 中键入内容。一个`ps -ef`显示容器内唯一运行的东西是`bash`和`ps`。键入`exit`离开容器的 bash shell。

```
bash-4.2$ which ruby
/opt/rh/rh-ruby22/root/usr/bin/ruby
bash-4.2$ ruby --version
ruby 2.2.2p95 (2015-04-13 revision 50295) [x86_64-linux]
bash-4.2$ pwd
/opt/app-root/src
bash-4.2$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
default      1     0  0 19:40 ?        00:00:00 /bin/bash
default     19     1  0 19:41 ?        00:00:00 ps -ef
bash-4.2$ exit
```

前面的`docker run`命令创建了一个容器来运行您的命令，保持任何状态，并将其与系统的其余部分隔离。您可以使用`docker ps`查看正在运行的容器列表。要查看所有已经创建的容器，包括那些已经退出的，使用`docker ps -a`。取决于你使用的是哪一个浮动文件，可能有许多其他的容器在运行，比如用于创建一个 OpenShift 环境的容器。

您可以使用`docker start`重新启动上面创建的容器。容器是通过名称来引用的。如果你不提供名字，Docker 会自动生成一个名字。一旦容器重新启动，`docker attach`将允许您与运行在其中的 shell 交互。请参见以下示例:

```
$ docker ps -a
CONTAINER ID        IMAGE                        COMMAND                  CREATED              STATUS                          PORTS               NAMES
8b08a6244a4c        rhscl/ruby-22-rhel7   "container-entrypoint"   3 minutes ago       Exited (0) 2 minutes ago                       determined_mayer

$ docker start determined_mayer
determined_mayer
$ docker attach determined_mayer
```

此时，您已经连接到容器内部正在运行的 shell。当你连接时，你不会看到命令提示符，所以按下回车键让它打印另一个。

```
bash-4.2$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
default      1     0  0 19:44 ?        00:00:00 /bin/bash
default     17     1  0 19:44 ?        00:00:00 ps -ef
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

在这一步中，您将创建一个使用 Ruby 作为 web 服务器的小型 Hello World 容器。一旦创建，容器就可以在安装了`docker`的其他系统上运行。您将需要使用您喜欢的编辑器在一个空目录中创建几个文件，包括一个描述如何构建容器映像的`Dockerfile`。

注意:你可以在你的主机系统上编辑文件，这些文件可以通过`vagrant rsync`同步到你的流浪箱。更多信息参见 *CDK 安装指南*中的*流浪同步文件夹*。

首先，创建一个空目录，然后用以下内容创建一个名为`Dockerfile`的文件，但是将`MAINTAINER`行改为包含您的姓名和电子邮件地址:

Dockerfile

```
FROM rhscl/ruby-22-rhel7

MAINTAINER Your Name "your-email@example.com"

EXPOSE 8000

COPY . /opt/app-root/src

CMD /bin/bash -c 'ruby hello-http.rb'
```

在与`Dockerfile`相同的目录下创建文件`hello-http.rb`:

你好-http.rb

```
require 'socket'

port = 8000
STDERR.print "Listening on port \n"

server = TCPServer.new('0.0.0.0', port)

loop do

  socket = server.accept
  request = socket.gets

  STDERR.puts request

  response = "Hello, Red Hat Developers World!\n"

  socket.print "HTTP/1.1 200 OK\r\n" +
               "Content-Type: text/plain\r\n" +
               "Connection: close\r\n"
  socket.print "\r\n"
  socket.print response

  socket.close
end
```

现在使用`docker build`构建容器映像。

`$ docker build -t *myname*/rubyweb .`

您可以看到使用以下命令创建的容器映像:

`$ docker images`

现在使用`docker run`运行容器。Ruby socket 服务器将创建一个小型 web 服务器，监听容器内部的端口 8000。`run`命令将主机上的端口 8000 映射到容器内的端口 8000。

`$ docker run -d -p 8000:8000 --name helloweb *myname*/rubyweb`

run 命令返回容器的唯一 ID，您可以忽略它。要检查容器是否正在运行，请使用`docker ps`。输出应该显示一个名为`helloweb`的容器，它正在运行您创建的`*myname*/rubyweb`容器映像。

```
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
c7885aa23773        myname/rubyweb    "container-entrypoint"   6 seconds ago       Up 4 seconds        0.0.0.0:8000->8000/tcp, 8080/tcp   helloweb
```

使用`curl`访问 Ruby web 服务器:

```
# curl http://localhost:8000/
Hello, Red Hat Developers World!
```

注意:您还应该能够从您的主机上的浏览器访问运行在您的容器中的 Ruby web 服务器。`rhel-ose/Vagrantfile`将无线箱的 IP 地址设置为 10.1.2.2。在您的主机系统上使用的 url 是`[http://10.1.2.2:8000/](http://10.1.2.2:8000/)`。

要查看运行容器中的日志，请使用`docker logs *<container-name>*`:

`$ docker logs helloweb`

完成后，停止正在运行的容器:

`$ docker stop helloweb`

`helloweb`容器将被保留，直到您用`docker rm`将其移除。你可以用`docker start helloweb`重启容器。注意:如果同名的容器已经存在，后续的`docker run`将产生一个错误。

您可以使用`docker inspect`查看有关容器的信息:

`$ docker inspect *myname*/rubyweb`

输出是一个易于阅读的 JSON 结构。 *Config* 部分包含容器运行时环境的细节，比如环境变量和默认命令。注意，容器配置中的许多信息都是从父容器继承的，在本例中是 Ruby 运行时容器。

最后，当您创建的应用程序容器映像准备就绪时，您可以通过将它们推送到公共或私有容器注册中心来分发它们。然后，您的容器将可以使用`docker pull`安装到其他系统上。

**想知道更多关于容器开发工具包的事情吗？**

### 成为红帽开发者:developers.redhat.com

Red Hat 提供专家资源和生态系统，帮助您提高工作效率并构建优秀的解决方案。在[developers.redhat.com](https://developers.redhat.com/)免费注册。

**关注红帽开发者博客**【https://developers.redhat.com/blog 

## 故障排除和常见问题

1.  我如何判断是否有一个容器映像有更新的 Ruby 版本？

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

    如果你是红帽技术合作伙伴，请访问[红帽技术合作伙伴连接](http://connect.redhat.com/)网站的[容器专区](https://access.redhat.com/articles/1483053)。

*Last updated: January 9, 2023*