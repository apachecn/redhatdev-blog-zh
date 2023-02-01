# Podman 简介(Red Hat Enterprise Linux 7.6 测试版)

> 原文：<https://developers.redhat.com/blog/2018/08/29/intro-to-podman>

几天前，红帽企业版 Linux (RHEL) 7.6 测试版发布了，我注意到的第一个新特性是 Podman。波德曼通过提供类似于 Docker 命令行的体验来补充 [Buildah](https://www.projectatomic.io/blog/2017/06/introducing-buildah/) 和 [Skopeo](https://www.projectatomic.io/blog/2016/07/working-with-containers-image-made-easy/) 的不足:允许用户运行独立的(非编排的)容器。而 Podman 不需要守护进程来运行容器和 pod，所以我们可以轻松地告别大胖守护进程。

Podman 实现了几乎所有的 Docker CLI 命令(当然，除了那些与 Docker Swarm 相关的命令)。对于容器编排，建议你看看 Kubernetes 和[红帽 OpenShift](http://openshift.com/) 。

Podman 只包含一个在命令行上运行的命令。没有后台守护进程在做事情，这意味着 Podman 可以通过`systemd`集成到系统服务中。

我们将介绍一些真实的例子，展示从 Docker CLI 过渡到 Podman 是多么容易。

## 波德曼装置

如果您运行的是 Red Hat Enterprise Linux 7.6 Beta，请遵循以下步骤。如果没有，你可以[用 Katacoda](http://katacoda.com/courses/containers-without-docker/running-containers-with-podman) 试试 Podman online。

您需要启用`extras` repo:

```
$ su -
# subscription-manager repos --enable rhel-7-server-extras-beta-rpms
```

**请注意:**撰写本文时，RHEL 7.6 仍处于测试阶段。一旦 GA 出现，请通过删除`-beta-`来更改存储库名称。

然后，启动正确的安装命令:

```
# yum -y install podman
```

这个命令将安装 Podman 及其依赖项:`atomic-registries`、`runC`、`skopeo-containers`和 SELinux 策略。

仅此而已。现在你可以和波德曼一起玩了。

## 命令行示例

### 经营一个 RHEL 集装箱

对于第一个例子，假设我们只想运行一个 RHEL 容器。我们在一个 RHEL 系统上，我们想运行一个 RHEL 容器，所以它应该工作:

```
[root@localhost ~]# docker run -it rhel sh
-bash: docker: command not found
```

如你所见，我的 RHEL 7.6 主机上没有`docker`命令。只需将`docker`命令替换为`podman`:

```
[root@localhost ~]# podman run -it rhel sh
Trying to pull registry.access.redhat.com/rhel:latest...Getting image source signatures
Copying blob sha256:367d845540573038025f445c654675aa63905ec8682938fb45bc00f40849c37b
71.46 MB / ? [------------=----------------------------------------------] 23s 
Copying blob sha256:b82a357e4f15fda58e9728fced8558704e3a2e1d100e93ac408edb45fe3a5cb9
1.27 KB / ? [----=--------------------------------------------------------] 0s 
Copying config sha256:f5ea21241da8d3bc1e92d08ca4888c2f91ed65280c66acdefbb6d2dba6cd0b29
6.52 KB / 6.52 KB [========================================================] 0s
Writing manifest to image destination
Storing signatures
sh-4.2#
```

我们现在有了我们的 RHEL 容器。让我们玩玩它，检查它的状态，然后删除它和它的源图像:

```
sh-4.2# ps ax
PID TTY STAT TIME COMMAND
1 pts/0 Ss 0:00 sh
10 pts/0 R+ 0:00 ps ax
sh-4.2# exit

[root@localhost ~]# podman ps -a
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
deda2991f9fd registry.access.redhat.com/rhel:latest sh 3 minutes ago Exited (0) Less than a second ago reverent_torvalds

[root@localhost ~]# podman rm deda2991f9fd
deda2991f9fd43400566abceaa917ecbd59a2e83354c5c9021ba1830a7ab196d

[root@localhost ~]# podman image rm rhel
f5ea21241da8d3bc1e92d08ca4888c2f91ed65280c66acdefbb6d2dba6cd0b29
```

如您所见，我们使用了与`docker`相同的语法。目前没有分歧。我没有检查波德曼文档，我立即开始工作！

### 运行一个 MariaDB 持久性容器

让我们向前看，尝试一个更复杂的测试:使用一些自定义变量运行 MariaDB 10.2，并尝试让它的“数据”持久化。

首先，让我们下载 MariaDB 容器映像并检查它的细节:

```
[root@localhost ~]# podman pull registry.access.redhat.com/rhscl/mariadb-102-rhel7Trying to pull registry.access.redhat.com/rhscl/mariadb-102-rhel7...Getting image source signatures
Copying blob sha256:367d845540573038025f445c654675aa63905ec8682938fb45bc00f40849c37b
71.46 MB / ? [------------=----------------------------------------------] 10s 
Copying blob sha256:b82a357e4f15fda58e9728fced8558704e3a2e1d100e93ac408edb45fe3a5cb9
1.27 KB / ? [----=--------------------------------------------------------] 0s 
Copying blob sha256:ddec0f65683ad89fc27298921921b2f8cbf57f674ed9eb71eef4e23a9dd9bbfe
6.40 MB / ? [--------------=----------------------------------------------] 1s 
Copying blob sha256:105cfda934d478ffbf65d74a89af55cc5de1d5bc94874c2d163c45e31a937047
58.25 MB / ? [-------------------------------------------=---------------] 10s 
Copying config sha256:7ac0a23445fec91d4b458f3062e64d1ca4af4755387604f8d8cbec08926867d7
6.79 KB / 6.79 KB [========================================================] 0s
Writing manifest to image destination
Storing signatures
7ac0a23445fec91d4b458f3062e64d1ca4af4755387604f8d8cbec08926867d7

[root@localhost ~]# podman images
REPOSITORY TAG IMAGE ID CREATED SIZE
registry.access.redhat.com/rhscl/mariadb-102-rhel7 latest 7ac0a23445fe 9 days ago 445MB

[root@localhost ~]# podman inspect 7ac0a23445fe
...
```

然后，我们可以设置一个文件夹，在启动容器后处理 MariaDB 的数据:

```
[root@localhost ~]# mkdir mysql-data
[root@localhost ~]# chown 27:27 mysql-data
```

**请注意:**“27”是将在容器中运行 MariaDB 进程的`mysql`用户的 ID。出于这个原因，我们必须允许它读取和写入目录。

最后，运行它:

```
[root@localhost ~]# podman run -d -v /root/mysql-data:/var/lib/mysql/data:Z -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 registry.access.redhat.com/rhscl/mariadb-102-rhel7
71da2bb210b36aaab28a2dc81b8e77da4e1024d1f2d025c0a7b97b075dec1425

[root@localhost ~]# podman ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
71da2bb210b3 registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest container-entrypoin... 3 seconds ago Up 3 seconds ago 0.0.0.0:3306->3306/udp, 0.0.0.0:3306->3306/tcp cranky_mahavira
```

如您所见，容器已经启动并运行，但是它在做什么呢？让我们检查一下:

```
[root@localhost ~]# podman logs 71da2bb210b3 | head
=> sourcing 20-validate-variables.sh ...
=> sourcing 25-validate-replication-variables.sh ...
=> sourcing 30-base-config.sh ...
---> 13:12:43 Processing basic MySQL configuration files ...
=> sourcing 60-replication-config.sh ...
=> sourcing 70-s2i-config.sh ...
---> 13:12:43 Processing additional arbitrary MySQL configuration provided by s2i ...
=> sourcing 40-paas.cnf ...
=> sourcing 50-my-tuning.cnf ...
---> 13:12:43 Initializing database ...
```

啊！它刚刚启动并初始化了它的数据库。让我们来玩玩它:

```
[root@localhost ~]# mysql --user=user --password=pass -h 127.0.0.1 -P 3306 -t
Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.2.8-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database |
+--------------------+
| db |
| information_schema |
| test |
+--------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> use test;
Database changed

MariaDB [test]> show tables;
Empty set (0.00 sec)
```

完美。现在我们将至少创建一个表，然后我们将终止容器:

```
MariaDB [db]> CREATE TABLE mytest (username VARCHAR(20), date DATETIME);
Query OK, 0 rows affected (0.02 sec)

MariaDB [db]> show tables;
+--------------+
| Tables_in_db |
+--------------+
| mytest |
+--------------+
1 row in set (0.00 sec)

MariaDB [db]> Bye

[root@localhost ~]# podman kill 71da2bb210b3
71da2bb210b36aaab28a2dc81b8e77da4e1024d1f2d025c0a7b97b075dec1425
```

检查文件夹的内容，我们可以看到数据仍然在那里，但是让我们启动一个新的容器来检查数据持久性:

```
[root@localhost ~]# ls -la mysql-data/
total 41024
drwxr-xr-x. 6 27 27 4096 Aug 24 09:12 .
dr-xr-x---. 4 root root 219 Aug 24 09:28 ..
-rw-rw----. 1 27 27 2 Aug 24 09:12 71da2bb210b3.pid
-rw-rw----. 1 27 27 16384 Aug 24 09:12 aria_log.00000001
-rw-rw----. 1 27 27 52 Aug 24 09:12 aria_log_control
drwx------. 2 27 27 56 Aug 24 09:27 db
-rw-rw----. 1 27 27 2799 Aug 24 09:12 ib_buffer_pool
-rw-rw----. 1 27 27 12582912 Aug 24 09:27 ibdata1
-rw-rw----. 1 27 27 8388608 Aug 24 09:27 ib_logfile0
-rw-rw----. 1 27 27 8388608 Aug 24 09:12 ib_logfile1
-rw-rw----. 1 27 27 12582912 Aug 24 09:12 ibtmp1
-rw-rw----. 1 27 27 0 Aug 24 09:12 multi-master.info
drwx------. 2 27 27 4096 Aug 24 09:12 mysql
-rw-r--r--. 1 27 27 14 Aug 24 09:12 mysql_upgrade_info
drwx------. 2 27 27 20 Aug 24 09:12 performance_schema
-rw-rw----. 1 27 27 24576 Aug 24 09:12 tc.log
drwx------. 2 27 27 6 Aug 24 09:12 test

[root@localhost ~]# podman run -d -v /root/mysql-data:/var/lib/mysql/data:Z -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 registry.access.redhat.com/rhscl/mariadb-102-rhel7
0364513f6b6ae1b86ea3752ec732bad757770ca14ec1f879e7487f3f4293004d

[root@localhost ~]# podman ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
0364513f6b6a registry.access.redhat.com/rhscl/mariadb-102-rhel7:latest container-entrypoin... 3 seconds ago Up 2 seconds ago 0.0.0.0:3306->3306/udp, 0.0.0.0:3306->3306/tcp heuristic_northcutt

[root@localhost ~]# mysql --user=user --password=pass -h 127.0.0.1 -P 3306 -t
Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.2.8-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> use db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [db]> show tables;
+--------------+
| Tables_in_db |
+--------------+
| mytest |
+--------------+
1 row in set (0.00 sec)

MariaDB [db]> Bye

[root@localhost ~]# podman kill 0364513f6b6a
0364513f6b6ae1b86ea3752ec732bad757770ca14ec1f879e7487f3f4293004d
```

太好了！MariaDB 的数据仍然在那里，新容器读取了它，并按照要求显示了一次。

### 通过 systemd 和 Podman 将容器作为系统服务进行管理

最后，我们将创建一个简单的`systemd`资源来处理之前创建的 MariaDB 容器。

首先，我们需要创建一个`systemd`资源文件来处理全新的容器服务:

```
[root@localhost ~]# cat /etc/systemd/system/mariadb-podman.service
[Unit]
Description=Custom MariaDB Podman Container
After=network.target

[Service]
Type=simple
TimeoutStartSec=5m
ExecStartPre=-/usr/bin/podman rm "mariadbpodman"

ExecStart=/usr/bin/podman run --name mariadbpodman -v /root/mysql-data:/var/lib/mysql/data:Z -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -p 3306:3306 registry.access.redhat.com/rhscl/mariadb-102-rhel7

ExecReload=-/usr/bin/podman stop "mariadbpodman"
ExecReload=-/usr/bin/podman rm "mariadbpodman"
ExecStop=-/usr/bin/podman stop "mariadbpodman"
Restart=always
RestartSec=30

[Install]
```

然后我们可以重新加载`systemd`目录并启动服务:

```
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl start mariadb-podman
[root@localhost ~]# systemctl status mariadb-podman
mariadb-podman.service - Custom MariaDB Podman Container
Loaded: loaded (/etc/systemd/system/mariadb-podman.service; static; vendor preset: disabled)
Active: active (running) since Fri 2018-08-24 10:14:36 EDT; 3s ago
Process: 19147 ExecStartPre=/usr/bin/podman rm mariadbpodman (code=exited, status=0/SUCCESS)
Main PID: 19172 (podman)
CGroup: /system.slice/mariadb-podman.service
└─19172 /usr/bin/podman run --name mariadbpodman -v /root/mysql-data:/var/lib/mysql/data:Z -e MYSQL_USER=user -e MYSQL_PASSWORD=pass -e MYSQL_DA...

Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140578968823552 [Note] InnoDB: Buffer pool(s) load completed at 180824 14:14:39
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Note] Plugin 'FEEDBACK' is disabled.
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Note] Server socket created on IP: '::'.
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Warning] 'user' entry 'root@71da2bb210b3' ignored in --sk...ve mode.
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Warning] 'user' entry '@71da2bb210b3' ignored in --skip-n...ve mode.
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Warning] 'proxies_priv' entry '@% root@71da2bb210b3' igno...ve mode.
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Note] Reading of all Master_info entries succeded
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Note] Added new Master_info '' to hash table
Aug 24 10:14:39 localhost.localdomain podman[19172]: 2018-08-24 14:14:39 140579889719488 [Note] /opt/rh/rh-mariadb102/root/usr/libexec/mysqld: read...ections.
Aug 24 10:14:39 localhost.localdomain podman[19172]: Version: '10.2.8-MariaDB' socket: '/var/lib/mysql/mysql.sock' port: 3306 MariaDB Server
Hint: Some lines were ellipsized, use -l to show in full.
[root@localhost ~]# systemctl stop mariadb-podman
[root@localhost ~]#
```

厉害！我们刚刚基于通过 Podman 管理的容器建立了一个定制的系统服务。

## 更多资源

你想要一个简单快捷的方法来试验波德曼吗？卡塔科达就是答案。Katacoda 是一个交互式学习和培训平台，让您在浏览器中使用真实环境学习新技术！

点击这里查看:[kata coda . com/courses/containers-with-docker/running-containers-with-podman](https://www.katacoda.com/courses/containers-without-docker/running-containers-with-podman)

要更好地了解波德曼，请看丹·沃尔什的两篇博客文章:

*   [重新引入波德曼](https://www.projectatomic.io/blog/2018/02/reintroduction-podman/)
*   [crictl vs poder man](https://blog.openshift.com/crictl-vs-podman/)

仅此而已！愿集装箱与你同在！:)

## 关于亚历山德罗

Alessandro Arrichiello 是 Red Hat Inc .的解决方案架构师。他从 14 岁开始就对 GNU/Linux 系统充满热情，这种热情一直持续到今天。他使用自动化企业 IT 的工具:配置管理和通过虚拟平台的持续集成。他现在致力于一个分布式云环境，涉及 PaaS (OpenShift)、IaaS (OpenStack)和流程管理(CloudForms)、容器构建、实例创建、HA 服务管理和工作流构建。

*Last updated: October 18, 2018*