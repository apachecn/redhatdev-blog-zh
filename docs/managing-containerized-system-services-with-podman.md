# 使用 Podman 管理集装箱化系统服务

> 原文：<https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman>

在本文中，我讨论了[容器](https://developers.redhat.com/blog/category/containers/)，但是从另一个角度来看它们。我们通常将容器称为开发新的云原生应用程序并与类似 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 的东西协调它们的最佳技术。回顾容器的起源，我们几乎忘记了容器是为了简化独立系统上的应用程序分发而诞生的。

在本文中，我们将讨论使用容器作为在[Red Hat Enterprise Linux](https://developers.redhat.com/products/rhel/)(RHEL)系统上安装应用程序和服务的完美媒介。使用容器并不复杂，我将展示如何在容器中运行 MariaDB、Apache HTTPD 和 Wordpress，同时通过 systemd 和`systemctl.`像管理任何其他服务一样管理这些容器

此外，我们将探索 Podman，这是 Red Hat 与 Fedora 社区共同开发的。如果你还不知道什么是波德曼，可以看看我之前的文章，[波德曼简介(Red Hat Enterprise Linux 7.6)](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/) 和汤姆·斯威尼的[没有守护进程的容器:RHEL 7.6 和 RHEL 8 Beta 中可用的波德曼和 Buildah】。](https://developers.redhat.com/blog/2018/11/20/buildah-podman-containers-without-daemons/)

## 红帽容器目录

首先，我们通过[红帽容器目录](https://access.redhat.com/containers/)【access.redhat.com/containers】来探索一下红帽企业 Linux 可用的容器:

![](img/2b59aaa049038b5034d5b26105080cda.png)

通过点击**浏览目录**，我们将访问 Red Hat 容器目录中可用的容器类别和产品的完整列表。

[![Exploring the available containers](img/5e1a49d32042751cded28c8a81af91a1.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/FireShot-Capture-010-Container-Catalog-Red-Hat_-https___access.redhat.com_containers__explore.png)

点击 **Red Hat Enterprise Linux** 将把我们带到 RHEL 部分，显示系统的所有可用容器映像:

[![Available RHEL containers](img/0fa3ec92a85e61971fd0f1944d6ec397.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/FireShot-Capture-009-Container-Catalog-Red-Hat-Customer-_-https___access.redhat.com_containe.png)

在撰写本文时，RHEL 类别中有超过 70 个容器映像，准备安装并用于 RHEL 7 系统。

因此，让我们选择一些容器映像，并在 Red Hat Enterprise Linux 7.6 系统上进行尝试。出于演示的目的，我们将尝试使用 Apache HTTPD + PHP 和 MariaDB 数据库来创建一个 Wordpress 博客。

## 安装集装箱服务

我们将首先安装我们的第一个容器化服务，用于建立一个 MariaDB 数据库，我们将需要这个数据库来托管 Wordpress 博客的数据。

作为安装容器化系统服务的先决条件，我们需要在 Red Hat Enterprise Linux 7 系统上安装名为 Podman 的实用程序:

```
[root@localhost ~]# subscription-manager repos --enable rhel-7-server-rpms --enable rhel-7-server-extras-rpms
[root@localhost ~]# yum install podman
```

正如我在上一篇文章中所解释的，Podman 通过提供类似于 Docker 命令行的体验来补充 Buildah 和 Skopeo:允许用户运行独立的(非编排的)容器。而 Podman 不需要守护进程来运行容器和 pod，所以我们可以轻松地告别大胖守护进程。

通过安装 Podman，你会发现 Docker 不再是一个必需的依赖项！

正如 Red Hat Container 目录的 MariaDB 页面所建议的，我们可以运行以下命令来完成任务(当然，我们将用`podman`替换`docker`):

```
[root@localhost ~]# podman pull registry.access.redhat.com/rhscl/mariadb-102-rhel7
Trying to pull registry.access.redhat.com/rhscl/mariadb-102-rhel7...Getting image source signatures
Copying blob sha256:9a1bea865f798d0e4f2359bd39ec69110369e3a1131aba6eb3cbf48707fdf92d
72.21 MB / 72.21 MB [======================================================] 9s
Copying blob sha256:602125c154e3e132db63d8e6479c5c93a64cbfd3a5ced509de73891ff7102643
1.21 KB / 1.21 KB [========================================================] 0s
Copying blob sha256:587a812f9444e67d0ca2750117dbff4c97dd83a07e6c8c0eb33b3b0b7487773f
6.47 MB / 6.47 MB [========================================================] 0s
Copying blob sha256:5756ac03faa5b5fb0ba7cc917cdb2db739922710f885916d32b2964223ce8268
58.82 MB / 58.82 MB [======================================================] 7s
Copying config sha256:346b261383972de6563d4140fb11e81c767e74ac529f4d734b7b35149a83a081
6.77 KB / 6.77 KB [========================================================] 0s
Writing manifest to image destination
Storing signatures
346b261383972de6563d4140fb11e81c767e74ac529f4d734b7b35149a83a081

[root@localhost ~]# podman images
REPOSITORY TAG IMAGE ID CREATED SIZE
registry.access.redhat.com/rhscl/mariadb-102-rhel7 latest 346b26138397 2 weeks ago 449MB
```

之后，我们可以查看 [Red Hat 容器目录页面](https://access.redhat.com/containers/?tab=tech-details&platform=systemimages#/registry.access.redhat.com/rhscl/mariadb-102-rhel7)以获得启动 MariaDB 容器映像所需变量的详细信息。

查看上一页，我们可以看到在*标签*下，有一个名为*用法*的标签，其中包含运行该容器映像的示例字符串:

```
usage docker run -d -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 rhscl/mariadb-102-rhel7
```

之后，我们需要一些关于我们的容器映像的其他信息:在容器中运行的“*用户 ID”和要附加的“*持久卷位置*”:*

```
[root@localhost ~]# podman inspect registry.access.redhat.com/rhscl/mariadb-102-rhel7 | grep User
"User": "27",
[root@localhost ~]# podman inspect registry.access.redhat.com/rhscl/mariadb-102-rhel7 | grep -A1 Volume 
"Volumes": {
"*/var/lib/mysql/data*": {}
[root@localhost ~]# podman inspect registry.access.redhat.com/rhscl/mariadb-102-rhel7 | grep -A1 ExposedPorts
"ExposedPorts": {
"*3306*/tcp": {}
```

此时，我们必须创建处理容器数据的目录；记住默认情况下容器是短暂的。然后，我们还要设置正确的权限:

```
[root@localhost ~]# mkdir -p /opt/var/lib/mysql/data
[root@localhost ~]# chown 27:27 /opt/var/lib/mysql/data
```

然后我们可以设置我们的`systemd`单元文件来处理数据库。我们将使用一个类似于前一篇文章中准备的单元文件:

```
[root@localhost ~]# cat /etc/systemd/system/mariadb-service.service
[Unit]
Description=Custom MariaDB Podman Container
After=network.target

[Service]
Type=simple
TimeoutStartSec=5m
ExecStartPre=-/usr/bin/podman rm "mariadb-service"

ExecStart=/usr/bin/podman run --name mariadb-service -v /opt/var/lib/mysql/data:/var/lib/mysql/data:Z -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=mysecret -e MYSQL_DATABASE=wordpress --net host registry.access.redhat.com/rhscl/mariadb-102-rhel7

ExecReload=-/usr/bin/podman stop "mariadb-service"
ExecReload=-/usr/bin/podman rm "mariadb-service"
ExecStop=-/usr/bin/podman stop "mariadb-service"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

让我们拆开我们的`ExecStart`命令并分析它是如何构建的:

*   `/usr/bin/podman run --name mariadb-service`说我们想要运行一个名为`mariadb-service`的容器。
*   *-* `v /opt/var/lib/mysql/data:/var/lib/mysql/data:Z`表示我们要将刚刚创建的数据目录映射到容器内的目录。`Z`选项通知 Podman 正确映射 SELinux 上下文以避免权限问题。
*   *-*-`e MYSQL_USER=wordpress -e MYSQL_PASSWORD=mysecret -e MYSQL_DATABASE=wordpress`标识与我们的 MariaDB 容器一起使用的附加环境变量。我们正在定义要使用的用户名、密码和数据库名称。
*   `--net host`将容器的网络映射到 RHEL 主机。
*   `registry.access.redhat.com/rhscl/mariadb-102-rhel7`指定要使用的容器图像。

我们现在可以重新加载`systemd`目录并启动服务:

```
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl start mariadb-service
[root@localhost ~]# systemctl status mariadb-service
mariadb-service.service - Custom MariaDB Podman Container
Loaded: loaded (/etc/systemd/system/mariadb-service.service; static; vendor preset: disabled)
Active: active (running) since Thu 2018-11-08 10:47:07 EST; 22s ago
Process: 16436 ExecStartPre=/usr/bin/podman rm mariadb-service ​(code=exited, status=0/SUCCESS)
Main PID: 16452 (podman)
CGroup: /system.slice/mariadb-service.service
└─16452 /usr/bin/podman run --name mariadb-service -v /opt/var/lib/mysql/data:/var/lib/mysql/data:Z -e MYSQL_USER=wordpress -e MYSQL_PASSWORD=mysecret -e MYSQL_DATABASE=wordpress --net host regist...

Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140276291061504 [Note] InnoDB: Buffer pool(s) load completed at 181108 15:47:14
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Note] Plugin 'FEEDBACK' is disabled.
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Note] Server socket created on IP: '::'.
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Warning] 'user' entry 'root@b75779533f08' ignored in --skip-name-resolve mode.
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Warning] 'user' entry '@b75779533f08' ignored in --skip-name-resolve mode.
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Warning] 'proxies_priv' entry '@% root@b75779533f08' ignored in --skip-name-resolve mode.
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Note] Reading of all Master_info entries succeded
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Note] Added new Master_info '' to hash table
Nov 08 10:47:14 localhost.localdomain podman[16452]: 2018-11-08 15:47:14 140277156538560 [Note] /opt/rh/rh-mariadb102/root/usr/libexec/mysqld: ready for connections.
Nov 08 10:47:14 localhost.localdomain podman[16452]: Version: '10.2.8-MariaDB' socket: '/var/lib/mysql/mysql.sock' port: 3306 MariaDB Server
```

完美！MariaDB 正在运行，所以我们现在可以开始为我们的 Wordpress 服务开发 Apache HTTPD + PHP 容器了。

首先，让我们从 Red Hat 容器目录中选择合适的容器:

```
[root@localhost ~]# podman pull registry.access.redhat.com/rhscl/php-71-rhel7
Trying to pull registry.access.redhat.com/rhscl/php-71-rhel7...Getting image source signatures
Skipping fetch of repeat blob sha256:9a1bea865f798d0e4f2359bd39ec69110369e3a1131aba6eb3cbf48707fdf92d
Skipping fetch of repeat blob sha256:602125c154e3e132db63d8e6479c5c93a64cbfd3a5ced509de73891ff7102643
Skipping fetch of repeat blob sha256:587a812f9444e67d0ca2750117dbff4c97dd83a07e6c8c0eb33b3b0b7487773f
Copying blob sha256:12829a4d5978f41e39c006c78f2ecfcd91011f55d7d8c9db223f9459db817e48
82.37 MB / 82.37 MB [=====================================================] 36s
Copying blob sha256:14726f0abe4534facebbfd6e3008e1405238e096b6f5ffd97b25f7574f472b0a
43.48 MB / 43.48 MB [======================================================] 5s
Copying config sha256:b3deb14c8f29008f6266a2754d04cea5892ccbe5ff77bdca07f285cd24e6e91b
9.11 KB / 9.11 KB [========================================================] 0s
Writing manifest to image destination
Storing signatures
b3deb14c8f29008f6266a2754d04cea5892ccbe5ff77bdca07f285cd24e6e91b
```

我们现在可以浏览此容器图像以获得一些细节:

```
[root@localhost ~]# podman inspect registry.access.redhat.com/rhscl/php-71-rhel7 | grep User
"User": "1001",
"User": "1001"
[root@localhost ~]# podman inspect registry.access.redhat.com/rhscl/php-71-rhel7 | grep -A1 Volume
[root@localhost ~]# podman inspect registry.access.redhat.com/rhscl/php-71-rhel7 | grep -A1 ExposedPorts
"ExposedPorts": {
"8080/tcp": {},
```

正如您在前面的命令中看到的，我们没有从容器详细信息中获得容量。你在问为什么吗？这是因为这个容器映像，即使它是 RHSCL(以前称为 Red Hat Software Collections)的一部分，也是为使用源到映像(S2I)构建器而准备的。要了解更多关于 S2I 构建器的信息，请查看其 GitHub 项目页面。

不幸的是，目前 S2I 实用程序严格依赖于 Docker，但是出于演示的目的，我们希望避免使用它..！

那么回到我们的问题，我们能做什么来猜测在我们的 PHP 容器上安装正确的文件夹？通过查看容器图像的所有环境变量，我们可以很容易地猜出正确的位置，在那里我们将找到`APP_DATA=/opt/app-root/src`。

因此，让我们用正确的权限创建这个目录；我们还将下载我们 Wordpress 服务的最新包:

```
[root@localhost ~]# mkdir -p /opt/app-root/src/
[root@localhost ~]# curl -o latest.tar.gz https://wordpress.org/latest.tar.gz
[root@localhost ~]# tar -vxf latest.tar.gz
[root@localhost ~]# mv wordpress/* /opt/app-root/src/
[root@localhost ~]# chown 1001 -R /opt/app-root/src
```

我们现在准备创建 Apache `http` + PHP `systemd`单元文件:

```
[root@localhost ~]# cat /etc/systemd/system/httpdphp-service.service
[Unit]
Description=Custom httpd + php Podman Container
After=mariadb-service.service

[Service]
Type=simple
TimeoutStartSec=30s
ExecStartPre=-/usr/bin/podman rm "httpdphp-service"

ExecStart=/usr/bin/podman run --name httpdphp-service -p 8080:8080 -v /opt/app-root/src:/opt/app-root/src:Z registry.access.redhat.com/rhscl/php-71-rhel7 /bin/sh -c /usr/libexec/s2i/run

ExecReload=-/usr/bin/podman stop "httpdphp-service"
ExecReload=-/usr/bin/podman rm "httpdphp-service"
ExecStop=-/usr/bin/podman stop "httpdphp-service"
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

然后我们需要重新加载`systemd`单元文件并启动我们最新的服务:

```
[root@localhost ~]# systemctl daemon-reload

[root@localhost ~]# systemctl start httpdphp-service

[root@localhost ~]# systemctl status httpdphp-service
httpdphp-service.service - Custom httpd + php Podman Container
Loaded: loaded (/etc/systemd/system/httpdphp-service.service; static; vendor preset: disabled)
Active: active (running) since Thu 2018-11-08 12:14:19 EST; 4s ago
Process: 18897 ExecStartPre=/usr/bin/podman rm httpdphp-service (code=exited, status=125)
Main PID: 18913 (podman)
CGroup: /system.slice/httpdphp-service.service
└─18913 /usr/bin/podman run --name httpdphp-service -p 8080:8080 -v /opt/app-root/src:/opt/app-root/src:Z registry.access.redhat.com/rhscl/php-71-rhel7 /bin/sh -c /usr/libexec/s2i/run

Nov 08 12:14:20 localhost.localdomain podman[18913]: => sourcing 50-mpm-tuning.conf ...
Nov 08 12:14:20 localhost.localdomain podman[18913]: => sourcing 40-ssl-certs.sh ...
Nov 08 12:14:20 localhost.localdomain podman[18913]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.12\. Set the 'ServerName' directive globall... this message
Nov 08 12:14:20 localhost.localdomain podman[18913]: [Thu Nov 08 17:14:20.925637 2018] [ssl:warn] [pid 1] AH01909: 10.88.0.12:8443:0 server certificate does NOT include an ID which matches the server name
Nov 08 12:14:20 localhost.localdomain podman[18913]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.88.0.12\. Set the 'ServerName' directive globall... this message
Nov 08 12:14:21 localhost.localdomain podman[18913]: [Thu Nov 08 17:14:21.017164 2018] [ssl:warn] [pid 1] AH01909: 10.88.0.12:8443:0 server certificate does NOT include an ID which matches the server name
Nov 08 12:14:21 localhost.localdomain podman[18913]: [Thu Nov 08 17:14:21.017380 2018] [http2:warn] [pid 1] AH10034: The mpm module (prefork.c) is not supported by mod_http2\. The mpm determines how things are ...
Nov 08 12:14:21 localhost.localdomain podman[18913]: [Thu Nov 08 17:14:21.018506 2018] [lbmethod_heartbeat:notice] [pid 1] AH02282: No slotmem from mod_heartmonitor
Nov 08 12:14:21 localhost.localdomain podman[18913]: [Thu Nov 08 17:14:21.101823 2018] [mpm_prefork:notice] [pid 1] AH00163: Apache/2.4.27 (Red Hat) OpenSSL/1.0.1e-fips configured -- resuming normal operations
Nov 08 12:14:21 localhost.localdomain podman[18913]: [Thu Nov 08 17:14:21.101849 2018] [core:notice] [pid 1] AH00094: Command line: 'httpd -D FOREGROUND'
Hint: Some lines were ellipsized, use -l to show in full.
```

让我们打开系统防火墙上的 8080 端口，连接到我们全新的 Wordpress 服务:

```
[root@localhost ~]# firewall-cmd --permanent --add-port=8080/tcp
[root@localhost ~]# firewall-cmd --add-port=8080/tcp
```

我们可以浏览我们的阿帕奇网络服务器:

[![Apache web server](img/254248bd7a2fb6926bde9e11dfa0cc3b.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/Screenshot-from-2018-11-08-18-16-06.png)

开始安装过程，并定义所有需要的细节:

[![Start the installation process](img/0278bc014f24a46b559e0957e277b1fa.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/Screenshot-from-2018-11-08-19-04-46.png)

最后，运行安装程序！

[![Run the installation](img/473137c17ea30970450fe2ab96f744a5.png)](https://developers.redhat.com/blog/wp-content/uploads/2018/11/Screenshot-from-2018-11-08-19-07-07.png)

最后，我们应该推出我们全新的博客，运行在 Apache `httpd` + PHP 上，由一个伟大的 MariaDB 数据库支持！

![](img/bb274f00da668ef21f7215592e227a16.png)

那都是乡亲；愿集装箱与你同在！

## 了解更多关于波德曼的信息

*   [Podman:在本地容器运行时管理 pod 和容器](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/)
*   [波德曼现在可以轻松过渡到 Kubernetes 和 CRI-O](https://developers.redhat.com/blog/2019/01/29/podman-kubernetes-yaml/)
*   没有守护进程的容器:RHEL 7.6 和 RHEL 8 测试版中的 Podman 和 Buildah
*   [pod man——下一代 Linux 容器工具](https://developers.redhat.com/articles/podman-next-generation-linux-container-tools/)
*   [pod man 简介(Red Hat Enterprise Linux 7.6 中的新功能)](https://developers.redhat.com/blog/2018/08/29/intro-to-podman/)

## 关于亚历山德罗

![Alessandro Arrichiello](img/7c535812708e4fd93e70794bffaf765b.png)

Alessandro Arrichiello 是 Red Hat Inc .的解决方案架构师，他从 14 岁开始就对 GNU/Linux 系统充满热情，这种热情一直持续到今天。他使用自动化企业 IT 的工具:配置管理和通过虚拟平台的持续集成。他现在致力于分布式云环境，涉及 PaaS (OpenShift)、IaaS (OpenStack)和流程管理(CloudForms)、容器构建、实例创建、HA 服务管理、工作流构建。

*Last updated: September 3, 2019*