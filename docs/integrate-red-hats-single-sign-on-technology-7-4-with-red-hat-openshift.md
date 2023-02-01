# 使用 Red Hat OpenShift 部署 Red Hat 的单点登录技术 7.4

> 原文：<https://developers.redhat.com/blog/2021/03/25/integrate-red-hats-single-sign-on-technology-7-4-with-red-hat-openshift>

在本文中，您将学习如何使用[红帽 OpenShift 4](https://developers.redhat.com/products/openshift/overview) 部署[红帽的单点登录技术](https://access.redhat.com/products/red-hat-single-sign-on) 7.4。对于这个集成，我们将使用 [PostgreSQL 数据库](https://www.postgresql.org/)。PostgreSQL 需要由外部网络文件系统(NFS)服务器分区提供的持久存储数据库。

## 先决条件

要遵循本文中的说明，您将需要开发环境中的以下组件:

*   具有领导者和追随者节点的 OpenShift 4 或更高版本的集群。
*   红帽的单点登录模板。
*   红帽开放移动数据基金会。

**注意**:出于本文的目的，我将引用领导者和跟随者节点，尽管代码输出使用了术语`master`和`worker`节点。

本文中描述的 OpenShift 存储配置是使用 NFS 的[持久存储，因为它对应于我们实验室中用于测试该设置的 OpenShift 存储配置。](https://docs.openshift.com/container-platform/4.8/storage/persistent_storage/persistent-storage-nfs.html)

OpenShift 还提供许多其他[持久存储配置](https://docs.openshift.com/container-platform/4.8/storage/understanding-persistent-storage.html)或块存储服务。然而，这些考虑超出了本文的范围。

## 为 OpenShift 4.5 集群创建持久存储

在这一节中，我们将描述使用 NFS 服务器在 OpenShift 4.5 上设置持久存储的不同步骤。

### 设置外部 NFS 服务器

NFS 允许远程主机通过网络装载文件系统，并与它们进行交互，就像它们是在本地装载的一样。这使系统管理员能够整合网络上集中服务器上的资源。有关 NFS 概念和基础的介绍，请参见[红帽企业 Linux 7](https://developers.redhat.com/products/rhel/overview) 文档中的[NFS 介绍](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-nfs)。

### 将 OpenShift 持久性存储映射到 NFS

要从 OpenShift 集群的 follower ( `worker`)节点访问 NFS 分区，必须手动将持久性存储映射到 NFS 分区。在本节中，您将执行以下操作:

*   获取节点列表。
*   访问追随者节点:
    *   ping NFS 服务器。
    *   挂载导出的 NFS 服务器分区。
    *   请验证 NFS 服务器上是否存在该文件。

#### 获取节点列表

要获取节点列表，请输入:

```
$ oc get nodes

```

请求的节点列表显示如下:

```
NAME STATUS ROLES AGE VERSION
master-0.example.com Ready master 81m v1.18.3+2cf11e2
worker-0.example.com Ready worker 72m v1.18.3+2cf11e2
worker-1.example.com Ready worker 72m v1.18.3+2cf11e2

```

#### 访问从动节点

要访问 follower 节点，使用`oc debug node`命令并键入`chroot /root`，如下所示:

```
$ oc debug node/worker-0.example.com

Starting pod/worker-example.com-debug ...

```

发出进一步命令前运行`chroot /host`:

```
sh-4.2# chroot /host
```

##### ping NFS 服务器

接下来，在调试模式下从 follower 节点 ping NFS 服务器:

```
sh-4.2#ping node-0.nfserver1.example.com

```

##### 挂载 NFS 分区

现在，从 follower 节点挂载 NFS 分区(仍处于调试模式):

```
sh-4.2#mount node-0.nfserver1.example.com:/persistent_volume1 /mnt

```

##### 验证 NFS 服务器上是否存在该文件

在调试模式下从 follower 节点创建一个虚拟文件:

```
sh-4.2#touch /mnt/test.txt

```

验证 NFS 服务器上是否存在该文件:

```
$ cd /persistent_volume1

```

```
$ ls -al
total 0
drwxrwxrwx. 2  root root  22 Sep 23 09:31 .
dr-xr-xr-x. 19 root root 276 Sep 23 08:37 ..
-rw-r--r--. 1 nfsnobody nfsnobody 0 Sep 23 09:31 test.txt

```

**注意**:您必须为集群中的每个从节点发出相同的命令序列。

## 永久卷存储

上一节展示了如何定义和挂载 NFS 分区。现在，您将使用 NFS 分区来定义和映射 OpenShift 持久性卷。步骤如下:

*   使永久卷在 NFS 服务器上可写。
*   将持久卷映射到 NFS 分区。
*   创建永久卷。

### 使永久卷可写

使永久卷在 NFS 服务器上可写:

```
$ chmod 777 /persistent_volume1
```

### 将持久卷映射到 NFS 分区

定义存储类并指定默认存储类。例如，下面的 YAML 定义了`slow`的`StorageClass`:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete

```

接下来，使存储类成为默认类:

```
$ oc create -f slow_sc.yaml 
$ oc patch storageclass slow -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'

```

**注意**:`StorageClass`是所有名称空间共有的。

### 创建永久卷

您可以从 OpenShift 管理控制台或 YAML 文件创建永久卷，如下所示:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: slow
  nfs:
    path: /persistent_volume2
    server: node-0.nfserver1.example.com

$ oc create -f pv.yaml
persistentvolume/example created

$ oc get pv
NAME CAPACITY ACCESS MODES RECLAIM POLICY STATUS CLAIM STORAGECLASS REASON AGE
example 5Gi RWO Retain Available slow 5s

```

## 在 OpenShift 集群上部署 SSO

接下来，您将在 OpenShift 集群上部署 Red Hat 的单点登录技术。步骤如下:

*   创建新项目。
*   下载`sso-74`模板。
*   自定义`sso74-ocp4-x509-postgresql-persistent`模板。

### 创建新项目

使用`oc new-project`命令创建一个新项目:

```
$ oc new-project sso-74
```

为 Red Hat 的单点登录技术 7.4 导入 OpenShift 映像:

```
$ oc -n openshift import-image rh-sso-7/sso74-openshift-rhel8:7.4 --from=registry.redhat.io/rh-sso-7/sso74-openshift-rhel8:7.4 --confirm

```

**注意**:如果需要删除并重新创建 SSO 项目，首先删除 secrets，这是项目特有的。

**注意**:命令(`oc -n openshift`)需要在特权集群用户(集群管理员)或者对 OpenShift 命名空间有写权限的用户下运行；不然就不行了。

### 下载 sso-74 模板

以下是可用模板的列表:

```
$ oc get templates -n openshift -o name | grep -o 'sso74.\+'

sso74-https
sso74-ocp4-x509-https
sso74-ocp4-x509-postgresql-persistent
sso74-postgresql
sso74-postgresql-persistent

```

### 自定义 SSO 74-oc P4-x509-PostgreSQL-persistent 模板

接下来，您将定制`sso74-ocp4-x509-postgresql-persistent`模板以允许 TLS 连接到持久 PostgreSQL 数据库:

```
$ oc process sso74-ocp4-x509-postgresql-persistent -n openshift SSO_ADMIN_USERNAME=admin SSO_ADMIN_PASSWORD=password -o yaml > my_sso74-x509-postgresql-persistent.yaml

```

## 控制手动设置 pod 副本计划

要设置 pod 副本调度，请在`sso`和
`sso-postgresql`部署配置的定义中更改`replicas`字段的设置。在更新的模板文件`my_sso74-x509-postgresql-persistent.yaml`中，将`sso`和`sso-postgresql`的两个副本设置为零(`0`)。

在每个部署配置中将副本设置为零(`0`)可让您手动控制初始 pod 卷展栏。如果这还不够，您还可以增加活性和就绪探测器的`initialDelaySeconds`值。以下是`sso`更新后的部署配置:

```
kind: DeploymentConfig
  metadata:
    labels:
      application: sso
      rhsso: 7.4.2.GA
      template: sso74-x509-postgresql-persistent
    name: sso
  spec:
    replicas: 0
    selector:
      deploymentConfig: sso

```

以下是`sso-postgresql`的更新配置:

```
metadata:
    labels:
      application: sso
      rhsso: 7.4.2.GA
      template: sso74-x509-postgresql-persistent
    name: sso-postgresql
  spec:
    replicas: 0
    selector:my_sso74-ocp4-x509-postgresql-persistent.yaml
      deploymentConfig: sso-postgresql

```

### 处理 YAML 模板

使用`oc create`命令处理 YAML 模板:

```
$ oc create -f my_sso74-x509-postgresql-persistent.yaml

service/sso created
service/sso-postgresql created
service/sso-ping created
route.route.openshift.io/sso created
deploymentconfig.apps.openshift.io/sso created
deploymentconfig.apps.openshift.io/sso-postgresql created
persistentvolumeclaim/sso-postgresql-claim created

```

### 升级 sso-postgresql pod

使用`oc scale`命令升级`sso-postgresql` pod:

```
$ oc scale --replicas=1 dc/sso-postgresql

```

**注意**:等待 PostgreSQL pod 到达`1/1`的就绪状态。这可能需要几分钟时间。

```
$ oc get pods
NAME                      READY   STATUS      RESTARTS   AGE
sso-1-deploy              0/1     Completed   0          10m
sso-postgresql-1-deploy   0/1     Completed   0          10m
sso-postgresql-1-fzgf7    1/1     Running     0          3m46s

```

当`sso-postgresql` pod 正确启动时，它会提供如下所示的日志输出:

```
pg_ctl -D /var/lib/pgsql/data/userdata -l logfile start waiting for server to start....2020-09-25 15:13:01.579 UTC [37] LOG: listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2020-09-25 15:13:01.588 UTC [37] LOG: listening on Unix socket "/tmp/.s.PGSQL.5432"
2020-09-25 15:13:01.631 UTC [37] LOG: redirecting log output to logging collector process
2020-09-25 15:13:01.631 UTC [37] HINT: Future log output will appear in directory "log".
done server started
/var/run/postgresql:5432 - accepting connections
=> sourcing /usr/share/container-scripts/postgresql/start/set_passwords.sh ...
ALTER ROLE
waiting for server to shut down.... done
server stopped
Starting server...
2020-09-25 15:13:06.147 UTC [1] LOG: listening on IPv4 address "0.0.0.0", port 5432
2020-09-25 15:13:06.147 UTC [1] LOG: listening on IPv6 address "::", port 5432
2020-09-25 15:13:06.157 UTC [1] LOG: listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2020-09-25 15:13:06.164 UTC [1] LOG: listening on Unix socket "/tmp/.s.PGSQL.5432"
2020-09-25 15:13:06.206 UTC [1] LOG: redirecting log output to logging collector process
2020-09-25 15:13:06.206 UTC [1] HINT: Future log output will appear in directory "log".

```

### 升级 sso pod

使用`oc scale`命令升级`sso` pod，如下所示:

```
$ oc scale --replicas=1 dc/sso
deploymentconfig.apps.openshift.io/sso

```

接下来，使用`oc get pods`命令，让 SSO pod 完全启动并运行。它到达`1/1`的就绪状态，如图所示:

```
$oc get pods
NAME                      READY   STATUS      RESTARTS   AGE
sso-1-d45k2               1/1     Running     0          52m
sso-1-deploy              0/1     Completed   0          63m
sso-postgresql-1-deploy   0/1     Completed   0          63m
sso-postgresql-1-fzgf7    1/1     Running     0          57m
```

## 测试

测试`oc status`命令包括:

```
$ oc status
In project sso-74 on server https://api.example.com:6443
svc/sso-ping (headless):8888
https://sso-sso-74.apps.example.com (reencrypt) (svc/sso)
  dc/sso deploys openshift/sso74-openshift-rhel8:7.4
    deployment #1 deployed about an hour ago - 1 pod

```

```
svc/sso-postgresql - 172.30.113.48:5432
  dc/sso-postgresql deploys openshift/postgresql:10
    deployment #1 deployed about an hour ago - 1 pod

```

访问网址[https://sso-sso-74.apps.example.com](https://sso-sso-74.apps.example.com)访问运行在 OpenShift 4.5 上的红帽单点登录技术 7.4 的管理员控制台。出现提示时，提供单一登录管理员用户名和密码。

## 结论

本文强调了在 OpenShift 上部署 Red Hat 单点登录技术 7.4 时要执行的基本步骤。在 OpenShift 上部署单点登录使得 OpenShift 的 SSO 特性开箱即用。例如，在水平扩展期间，通过向 OpenShift 部署添加新的单点登录 pod，可以非常容易地增加您的工作负载容量。

*Last updated: October 7, 2022*