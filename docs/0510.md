# Docker 用户的 Red Hat 通用基本图像

> 原文：<https://developers.redhat.com/blog/2020/03/24/red-hat-universal-base-images-for-docker-users>

[Red Hat Universal Base Images(UBIs)](https://developers.redhat.com/products/rhel/ubi/)允许在 Windows 和 Mac 平台上使用 Docker 的开发人员利用大型 Red Hat 生态系统的优势。本文演示了如何在非 Red Hat 系统(如 Windows 或 Mac 工作站)中使用 Docker 的 Red Hat Universal Base 映像。

## Red Hat Enterprise Linux 和 Docker

当 Red Hat Enterprise Linux (RHEL) 8 在大约一年前发布时，它附带了许多与容器相关的新特性。其中最大的是新的容器工具(Podman、Buildah 和 skopeo)和新的 Red Hat Universal Base Images。由于 RHEL 8 放弃了对 Docker 工具集的支持，也出现了混乱。一些开发人员认为他们不能再使用 Docker 了，他们要么迁移到红帽生态系统 Linux 系统，比如 CentOS，要么远离红帽客户。

这种情况与事实相去甚远，因为容器不再仅仅与码头有关。容器运行时、容器映像、注册服务器和其他与 Linux 容器生态系统相关的技术现在已经由[开放容器倡议](https://www.opencontainers.org/) (OCI)标准化了。多亏了 OCI，您可以使用一个工具开发一个容器，然后使用另一个工具运行同一个容器。例如，Red Hat 在 RHEL 8 上使用 Buildah 构建一个容器映像，然后您在 Windows 系统上使用 Docker 运行该容器映像。

另一个例子是在 Mac 系统上使用 Docker 构建一个容器映像，然后在 Red Hat Enterprise Linux 8 服务器上使用 Podman 运行该容器映像。

## 介绍 Red Hat 通用基本图像

Red Hat Universal Base images 允许商业和开放源码开发者构建基于 RHEL 的容器，而不需要他们或他们的用户是 RHEL 订户。开源开发者需要能够在任何地方运行和共享他们的应用程序，不受用户协议的限制。虽然 Red Hat Developer 订阅很有用，也很受欢迎，但它并不是最适合他们的。

一些商业开发者想同时瞄准红帽客户和非红帽客户。他们的 Red Hat 客户希望能够充分利用 Red Hat 支持服务，这意味着运行基于 RHEL 的容器映像。因此，这些开发人员必须构建两个版本的容器映像，一个基于 Red Hat Enterprise Linux，另一个基于其他东西，或者他们必须拒绝他们的客户依赖 Red Hat 的支持服务。

商业和开源开发者可以使用 UBI 从 RHEL 包中构建容器映像，并从 Red Hat 对性能和安全问题的修复中受益。同时，使用 UBIs 不需要任何红帽订阅的权利，甚至不需要免费的红帽开发者计划订阅。

### 通用基础映像和软件包

UBIs 提供了一组基本容器映像来构建您的应用程序映像。其中一些映像只包含基本操作系统。其他的是运行时映像，它们为运行时提供预先集成的依赖关系。

UBIs 还提供了一组 Yum 存储库，其中包括 Red Hat Enterprise Linux 包的子集。并非整个 RHEL 都是 UBI 库的一部分。例如，ubi 不包括与低级网络和存储服务器相关的包。

### 探索通用基础映像

查看 UBIs 提供了什么的最好方法是通过一些搜索。在下面的例子中，提示是`$`，可能是在运行 Docker Desktop 的 Mac 系统上。同样的提示也可以出现在使用 Docker 工具的 Windows Home 系统的 Cygwin 终端上，或者出现在使用 Docker 桌面的 Windows Professional 系统上。它甚至可以在非 Red Hat Linux 发行版上。

首先，在 [Red Hat 容器注册表](https://registry.access.redhat.com)上搜索所有 UBI 基本映像:

```
$ docker search registry.access.redhat.com/ubi
NAME                                          DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
jboss-eap-7/eap72-openjdk11-openshift-rhel8   JBoss EAP 7.2 OpenJDK 11 Container Image for…   0      
ubi8/ubi-init                                 Provides the latest release of the Red Hat U…   0      
ubi8/ubi                                      Provides the latest release of the Red Hat U…   0      
ubi8/ubi-minimal                              Provides the latest release of the Minimal R…   0      
ubi7/ubi-init                                 The Universal Base Image Init is designed to…   0      
ubi7/ubi                                      The Universal Base Image is designed and eng…   0      
ubi7/ubi-minimal                              The Universal Base Image Init is designed to…   0

```

Red Hat 容器注册中心是一个公共注册中心，托管所有 UBI 容器映像。这些相同的图像也可以从基于 [Red Hat terms 的容器注册表](https://registry.redhat.io)中获得。该注册服务器要求您进行身份验证，以证明您拥有有效的 Red Hat 订阅。在本文中，为了使用方便，我将坚持使用 Red Hat 的公共注册表。

现在，启动一个运行交互式 shell 的 UBI 测试容器:

```
$ docker run -it --name test registry.access.redhat.com/ubi8/ubi:8.1 bash
Unable to find image 'registry.access.redhat.com/ubi8/ubi:8.1' locally
8.1: Pulling from ubi8/ubi
eae5d284042d: Pull complete
ff6f434a470a: Pull complete
Digest: sha256:b6ae810838a1a105b568e5b438a4379ac5e06ee8df1c11d71772f8708180ffcc
Status: Downloaded newer image for registry.access.redhat.com/ubi8/ubi:8.1
[root@de1d73d88418 /]#
```

在测试容器中，列出其预配置的 Yum 存储库:

```
[root@de1d73d88418 /]# yum repolist
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Red Hat Universal Base Image 8 (RPMs) - BaseOS                                                      123 kB/s | 760 kB     00:06    
Red Hat Universal Base Image 8 (RPMs) - AppStream                                                   301 kB/s | 3.3 MB     00:11    
Red Hat Universal Base Image 8 (RPMs) - CodeReady Builder                                           2.2 kB/s | 9.1 kB     00:04    
repo id                                       repo name                                                                       status
ubi-8-appstream                               Red Hat Universal Base Image 8 (RPMs) - AppStream                               819
ubi-8-baseos                                  Red Hat Universal Base Image 8 (RPMs) - BaseOS                                  664
ubi-8-codeready-builder                       Red Hat Universal Base Image 8 (RPMs) - CodeReady Builder                        12
```

**注意:**你可以放心地忽略关于 Red Hat 的订阅管理器的 Yum 消息。UBI 映像也是为 Red Hat 订户设计的，允许他们从 Red Hat Enterprise Linux 添加需要有效授权的包。

您还可以从这些存储库中搜索单个可用的包。以下示例搜索 Nginx web 服务器包:

```
[root@de1d73d88418 /]# yum search nginx
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:06 ago on Mon Feb 24 23:37:46 2020.
=================================================== Name Exactly Matched: nginx ====================================================
nginx.x86_64 : A high performance web server and reverse proxy server
================================================== Name & Summary Matched: nginx ===================================================
nginx-mod-mail.x86_64 : Nginx mail modules
nginx-mod-stream.x86_64 : Nginx stream modules
nginx-mod-http-perl.x86_64 : Nginx HTTP perl module
nginx-mod-http-xslt-filter.x86_64 : Nginx XSLT module
nginx-mod-http-image-filter.x86_64 : Nginx HTTP image filter module
nginx-filesystem.noarch : The basic directory layout for the Nginx server
nginx-all-modules.noarch : A meta package that installs all available Nginx modules
```

最后，离开测试容器，停止，并将其移除:

```
[root@de1d73d88418 /]# exit
$ docker stop test
test
$ docker rm test
test
```

如你所见，使用 Docker 运行 UBIs 就可以了。但是，如何基于通用基础映像构建容器映像呢？

# 一个 PHP 应用程序的样例 docker 文件

下面的清单显示了一个 PHP 应用程序的 docker 文件示例:

```
FROM registry.access.redhat.com/ubi8/ubi:8.1

RUN yum --disableplugin=subscription-manager -y module enable php:7.3 \
  && yum --disableplugin=subscription-manager -y install httpd php \
  && yum --disableplugin=subscription-manager clean all

ADD index.php /var/www/html

RUN sed -i 's/Listen 80/Listen 8080/' /etc/httpd/conf/httpd.conf \
  && sed -i 's/listen.acl_users = apache,nginx/listen.acl_users =/' /etc/php-fpm.d/www.conf \
  && mkdir /run/php-fpm \
  && chgrp -R 0 /var/log/httpd /var/run/httpd /run/php-fpm \
  && chmod -R g=u /var/log/httpd /var/run/httpd /run/php-fpm

EXPOSE 8080
USER 1001
CMD php-fpm & httpd -D FOREGROUND
```

第二个`RUN`指令允许镜像在 OpenShift 的默认安全策略和无根 podman 下不变地运行。否则，该映像将需要以 root 用户身份运行，这是普通 OpenShift 用户无法做到的。

对于`index.php`文件，您还需要一个“hello，world”风格的 PHP 页面:

```
<html>
<body>
<?php print "Hello, world!\n" ?>
</body>
</html>
```

现在构建示例图像:

```
$  docker build -t php-hello .
Sending build context to docker daemon  3.584kB
Step 1/7 : FROM registry.access.redhat.com/ubi8/ubi:8.1
 ---> fd73e6738a95
...
Successfully tagged php-hello:latest
```

并从示例图像启动一个容器:

```
$ docker run --name hello -p 8080:8080 -d php-hello
```

最后，打开 web 浏览器并访问`localhost:8080`来查看由您的示例容器返回的`Hello, world!`消息。当您对结果满意时，停止并删除样品容器:

```
$ docker stop hello
hello
$ docker rm hello
hello
```

如您所见，在 Docker 中使用通用基础映像并没有什么不寻常的。他们只是工作。

*2020 年 9 月 14 日更新:修复了 docker 文件样例，可以与 RHEL 8.1+和 Fedora 的无根 podman 一起工作。改变不影响在 Docker 和早期 OpenShift 4.x 下运行*

# 了解有关 UBIs 的更多信息

*   [通用基础映像(UBI):来自 Red Hat 客户门户的映像、存储库和包](https://access.redhat.com/articles/4238681)。
*   Red Hat Developer 的 Red Hat Universal Base Images(UBI)，该网站还提供了 [UBI FAQ](https://developers.redhat.com/articles/ubi-faq/) 以及其他资源。
*   [使用 Red Hat Universal Base Image 与 Azure Pipelines 和 Red Hat Quay.io](https://www.redhat.com/en/blog/using-red-hat-universal-base-image-azure-pipelines-and-red-hat-quayio) 来自 Red Hat 博客，这是使用 UBIs 与非 Red Hat 工具的一个很好的例子。
*   [红帽合作伙伴现在如何自由地重新分发更多的 RHEL 包](https://developers.redhat.com/blog/2020/02/26/red-hat-simplifies-container-dev-and-redistribution-rhel-packages/)。

*Last updated: April 1, 2022*