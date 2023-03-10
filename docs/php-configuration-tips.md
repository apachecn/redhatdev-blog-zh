# PHP 配置技巧

> 原文：<https://developers.redhat.com/blog/2017/10/25/php-configuration-tips>

**RHEL 7** 提供了 Apache HTTP Server 版本 2.4 和 PHP 版本 5.4。

Apache httpd 和 PHP 使用的最常见配置，但是这有一些限制和缺点:

*   可以使用 mod_php 的一个 PHP 版本
*   mod_php 运行在 httpd 进程中，没有任何隔离
*   mod _ phpis 仅支持 prefork MPM

本文将解释如何配置 Apache httpd 来使用 **FastCGI** 协议将 PHP 脚本的执行委托给后端，如何使用更**的最新 PHP 版本**，如何运行**的多个 PHP 版本**，以及如何提高 Apache httpd **的性能**。

RHEL 的 Apache httpd 包提供了使用这种配置所需的所有特性。

## 1.切换到 php-fpm

### 1.1.删除修改 php

建议删除或禁用 mod_php，以减少每个 httpd 进程的内存占用。

您可以删除 **php** 包，它只提供这个模块:

```
 yum remove php
```

或者通过注释掉/etc/httpd/conf . modules . d/10-PHP . conf 中的 LoadModule 指令来禁用它。

```
 # disabled # LoadModule php5_module modules/libphp5.so
```

### 1.2.安装 php-fpm

现在您可以安装 php-fpm 并启用它的服务。

```
 yum install php-fpm
 systemctl start php-fpm
 systemctl enable php-fpm
```

注意: **php-fpm** 包在**可选**通道中有，该通道必须开启。

要配置 PHP 脚本执行，请编辑或创建/etc/httpd/conf.d/php.conf 文件:

以下行阻止 Web 客户端查看. user.ini 文件。

```
  <Files ".user.ini">
    Require all denied
  </Files>
```

允许 php 处理多视图:

```
  AddType text/html .php
```

将 index.php 添加到将作为目录索引的文件列表中:

```
  DirectoryIndex index.php
```

在下面一行，启用 http 授权头:

```
  SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
```

将 PHP 脚本执行重定向到 FPM 后端。

```
  <FilesMatch \.php$>
    SetHandler "proxy:fcgi://127.0.0.1:9000"
  </FilesMatch>
```

如果你在这个文件中有一些 **php_value** 指令，你需要删除它们，它们只适用于 mod_php。

现在您可以(重新)启动 web 服务器和一个简单的 PHP 测试页面；

```
<?php phpinfo();
```

它会显示你现在正在通过 FatCGI 后端运行 PHP。

```
PHP Version 5.4.16
Server API= FPM/FastCGI
```

### 1.3.PHP 调优

主要的 FPM 配置文件是/etc/php-fpm.conf，其中有很多解释每个选项的注释。

FPM 可以运行各种各样的池，每一个都运行带有不同选项的 PHP 脚本，默认的池(www)配置文件是/etc/php-fpm.d/www.conf，其中也有很多注释。

#### 1.3.1.php_value，php-flag

可以使用 *php_value、php_admin_value、* *php_flag* 和 *php_admin_flag* 指令设置 PHP 选项:

*   用 mod_php，在 Apache httpd 配置文件中。
*   与 FPM，在池配置文件。

#### 1.3.2.。htaccess

可以在特定目录中设置附加选项:

*   用 mod_php，用一个**。htaccess** 文件。
*   对于 FPM，使用一个 **.user.ini** 文件(不需要 php_*关键字)。

#### 1.3.3.过程调整

FPM 作为守护程序运行，启动各种进程，以便能够同时处理各种请求，并提供各种模式:

*   pm = ondemand，child 只在连接打开时启动，空闲时停止，适合开发环境
*   pm =动态，一组空闲的进程总是在运行，如果需要可以启动更多的进程，适合生产
*   pm =静态，一套固定的流程总是在运行，适合生产，可以更好地获得性能

### 1.4.Apache HTTP 服务器调优

#### 1.4.1.螺纹 MPM

默认情况下，Apache HTTP 服务器使用一组进程来管理传入的请求(prefork MPM)。

由于我们现在不使用 mod_php，我们可以切换到一个线程化的 MPM(一个事件的工作线程),这样一组线程将管理请求，减少运行进程的数量和内存占用，并提高性能，尤其是在处理大量静态文件时。

在/etc/httpd/conf . modules . d/00-MPM . conf 配置文件中切换使用的 MPM。

```
  # disabled # LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
  # disabled # LoadModule mpm_worker_module modules/mod_mpm_worker.so
  LoadModule mpm_event_module modules/mod_mpm_event.so
```

#### 1.4.2.Unix 域套接字

默认情况下，FPM 监听网络套接字上的传入请求，但可以使用 Unix 域套接字，这可以稍微提高性能。

在 **FPM** 池配置:

```
  listen = /run/php-fpm/www.sock
  listen.owner = apache
  listen.mode = 0660
```

在**阿帕奇**T2【httpd】配置中:

```
  SetHandler "proxy:unix:/run/php-fpm/www.sock|fcgi://localhost"
```

#### 1.4.2.独立的前端和后端服务器

默认情况下，FPM 监听本地网络套接字上的传入请求。当然，它可以运行在单独的服务器、另一个虚拟机或容器(docker 实例)上

在 **FPM** 池配置:

```
  listen = 10.0.0.2:9000
  listen.allowed_clients = 10.0.0.1
```

在 **Apache httpd** 配置中:

```
  SetHandler "proxy:fcgi://10.0.0.2:9000"
```

#### 多个 php 后端

为了能够处理更多的并发请求，我们可能希望在各种 PHP 后端之间平衡负载，这很容易。

**Apache httpd** 配置示例，有 3 个后端:

```
  # Load balancer creation
  <Proxy balancer://phpfpmlb>
    BalancerMember fcgi://10.0.0.2:9000
    BalancerMember fcgi://10.0.0.3:9000
    BalancerMember fcgi://10.0.0.4:9000
  </Proxy>
  # Redirect PHP execution to the balancer
  <FilesMatch \.php$>
    SetHandler "proxy:balancer://phpfpmlb"
 </FilesMatch>
```

## 2.运行最新的 PHP 版本

RHEL 提供了 PHP 版本 **5.4** ，这是 RHEL-7 发布时的最新版本，但对于最近的一些项目来说可能太旧了。

PHP 版本 **5.6** 和 **7.0** 今天在 RHEL 作为红帽软件集合( [RHSCL](https://access.redhat.com/documentation/en-US/Red_Hat_Software_Collections/2/html/2.4_Release_Notes/index.html) )的一部分得到支持，PHP 版本 7.1 目前正与 RHSCL 的 [3.0 更新](https://access.redhat.com/documentation/en-US/Red_Hat_Software_Collections/3-Beta/html/3.0_Release_Notes/index.html)一起进行 Beta 测试。

在下面的例子中，我们将使用 7.0 版本，但是它也可以用于其他可用的版本。

### **2.1。安装**

启用 RHSCL 通道后，安装软件集合:

```
  yum install rh-php70
```

为此版本安装 FPM 服务:

```
  yum install rh-php70-php-fpm
```

安装任何需要的附加扩展:

```
  yum install rh-php70-php-mbstring rh-php70-php-pgsql rh-php70-php-opcache
```

**提示**:比较可用扩展列表，确保所需的一切都可用。

```
  php --modules | tee /tmp/54
  scl enable rh-php70 'php --modules' | tee /tmp/70
  diff /tmp/54 /tmp/70
```

**提示**:永远不要依赖包名，而要使用扩展名(例如 php-mysqli 或 rh-php70-php-simplexml)，因为包的布局可能会随着版本的不同而变化。

### 2.2.切换到更新的 PHP 版本

运行 FPM 时，这就像停止旧版本服务并启动新版本服务一样简单:

```
  systemctl stop  php-fpm
  systemctl start rh-php70-php-fpm
```

### 2.3.附加包

软件集合提供了与 RHEL 标准软件包相同的 PHP 扩展集。

随着用户习惯于寻找一些额外的扩展，在[https://www.softwarecollections.org/](https://fedoraproject.org/wiki/EPEL)资源库中，可以在社区 **centos-sclo-sclo** 资源库中找到额外的扩展，更多信息请在[上搜索 **sclo-php** 。](https://www.softwarecollections.org/en/scls/?search=sclo-php)

## 3.运行多个版本的 PHP

由于 PHP 的执行是使用 **SetHandler** 指令重定向到 FastCGI 服务的，所以这可以根据 vhost、项目或目录进行设置。

在下面的例子中，我们将同时运行基础系统中的 PHP 5.4 版本(对于一些遗留应用程序，已经在上面进行了配置)和 PHP 7.1 版本。

### 3.1.装置

启用 RHSCL beta 通道后，安装软件集合:

```
  yum install rh-php71 rh-php71-php-fpm rh-php71-php-mbstring rh-php71-php-opcache ...
```

在/etc/opt/RH/RH-PHP 71/php-fpm . d/www . conf 中，将 FPM 配置为侦听不同于默认 PHP-fpm 服务使用的端口

```
  listen = 127.0.0.1:9071
```

确保此端口未被 SELinux 阻塞:

```
  semanage port -a -t http_port_t -p tcp 9071
```

启动服务:

```
  systemctl start rh-php71-php-fpm
```

现在，可以从 Apache httpd 配置文件中为每个目录选择 PHP 版本。

示例:

```
  # Use PHP 7.1 by default
  <FilesMatch \.php$>
    SetHandler "proxy:fcgi://127.0.0.1:9071"
  </FilesMatch>
  # Some legacy application use PHP 5.4
  <Directory /var/www/html/old>
    <FilesMatch \.php$>
      SetHandler "proxy:fcgi://127.0.0.1:9000"
    </FilesMatch>
  </Directory>
```

## 4.结论

我希望这篇小文章已经展示了 PHP 脚本切换到 FPM 的各种好处:

*   前端(httpd)和后端(fpm)之间的进程隔离
*   性能改进
*   运行现代 PHP 版本
*   运行多个 PHP 版本

*Last updated: October 18, 2018*