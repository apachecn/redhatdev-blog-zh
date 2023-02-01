# 配置容器化服务

> 原文：<https://developers.redhat.com/blog/2017/07/10/configuring-containerized-services>

我假设您已经尝试过运行一些多容器应用程序的例子。假设我们有一个由以下内容组成的应用程序:

*   网络服务
*   数据库ˌ资料库
*   键值存储
*   工人

现在让我们把重点放在数据库上。数据库的容器映像通常带有一种通过环境变量配置它们的简单方法。这种方法的伟大之处在于它的易用性，例如，让我们以我们的 [RHSCL PostgreSQL 9.5 容器映像](https://access.redhat.com/containers/#/registry.access.redhat.com/rhscl/postgresql-95-rhel7)为例。如果您试图像这样运行它，它会拒绝运行，而是指导您应该如何运行容器:

```
$ docker run registry.access.redhat.com/rhscl/postgresql-95-rhel7:latest
You must either specify the following environment variables:
  POSTGRESQL_USER (regex: '^[a-zA-Z_][a-zA-Z0-9_]*$')
  POSTGRESQL_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:]+$')
  POSTGRESQL_DATABASE (regex: '^[a-zA-Z_][a-zA-Z0-9_]*$')
Or the following environment variable:
  POSTGRESQL_ADMIN_PASSWORD (regex: '^[a-zA-Z0-9_~!@#$%^&*()-=<>,.?;:]+$')
Or both.
Optional settings:
  POSTGRESQL_MAX_CONNECTIONS (default: 100)
  POSTGRESQL_MAX_PREPARED_TRANSACTIONS (default: 0)
  POSTGRESQL_SHARED_BUFFERS (default: 32MB)

For more information see /usr/share/container-scripts/postgresql/README.md
within the container or visit https://github.com/openshift/postgresql.
```

整洁！所以让我们传递环境变量:

```
$ docker run --rm \
    -e POSTGRESQL_USER=test_user \
    -e POSTGRESQL_PASSWORD=secret \
    -e POSTGRESQL_DATABASE=test_database \
    --name=pg \
    registry.access.redhat.com/rhscl/postgresql-95-rhel7:latest
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.utf8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/pgsql/data/userdata ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
creating template1 database in /var/lib/pgsql/data/userdata/base/1 ... ok
initializing pg_authid ... ok
initializing dependencies ... ok
creating system views ... ok
loading system objects' descriptions ... ok
creating collations ... ok
creating conversions ... ok
creating dictionaries ... ok
setting privileges on built-in objects ... ok
creating information schema ... ok
loading PL/pgSQL server-side language ... ok
vacuuming database template1 ... ok
copying template1 to template0 ... ok
copying template1 to postgres ... ok
syncing data to disk ... ok

output shortened

LOG: redirecting log output to logging collector process
HINT: Future log output will appear in directory "pg_log".
```

它在跑！

如果我们想开始配置服务，我们可以使用一堆环境变量，比如`POSTGRESQL_SHARED_BUFFERS`。这就是我们可能碰壁的地方:我们如何配置其他参数？

首先要注意的是，如果图片清楚地记录了如何更改内部运行的容器化服务的配置，那就太棒了。

同时，让我们看看配置容器化服务的最佳方式。

## 重叠配置

在此方法中，您应该通过在现有容器映像的基础上应用配置更改来创建新的容器映像。让我们用之前提到的 PostgreSQL 容器映像来尝试一下。

首先要做的是从运行容器中复制现有的配置文件模板`openshift-custom-postgresql.conf.template`:

```
$ docker cp pg:/usr/share/container-scripts/postgresql/openshift-custom-postgresql.conf.template ./openshift-custom-postgresql.conf.template
```

我们现在应该编辑模板文件并生成自定义 postgresql 容器映像。

```
#
# Custom OpenShift configuration.
#
# NOTE: This file is rewritten every time the container is started!
# Changes to this file will be overwritten.
#

# Listen on all interfaces.
listen_addresses = '*'

# Determines the maximum number of concurrent connections to the database server. Default: 100
max_connections = ${POSTGRESQL_MAX_CONNECTIONS}

# Allow each connection to use a prepared transaction.
max_prepared_transactions = ${POSTGRESQL_MAX_PREPARED_TRANSACTIONS}

# Sets the amount of memory the database server uses for shared memory buffers. Default: 32MB
shared_buffers = ${POSTGRESQL_SHARED_BUFFERS}

# Sets the planner's assumption about the effective size of the disk cache that is available to a single query.
effective_cache_size = ${POSTGRESQL_EFFECTIVE_CACHE_SIZE}

# Let’s increase work_mem so most operations are performed in memory
work_mem = 64MB

```

我们在模板中添加了最后两行。

现在是构建映像的时候了，我们将使用这个 docker 文件:(我们更新了标签，所以很明显新构建的映像不是正式的 RHSCL 映像，而是基于它的变体)

```
FROM registry.access.redhat.com/rhscl/postgresql-95-rhel7:latest
LABEL name="tomastomecek/postgresql" \
      vendor="Tomas Tomecek"
COPY ./openshift-custom-postgresql.conf.template /usr/share/container-scripts/postgresql/

```

让我们建造...

```
$ docker build --tag=tomastomecek/postgresql .
```

然后跑...

```
$ docker run --rm \
    -e POSTGRESQL_USER=pg_test \
    -e POSTGRESQL_PASSWORD=secret \
    -e POSTGRESQL_DATABASE=test_db \
    --name pg tomastomecek/postgresql
```

并检查`work_mem`属性是否被更改:

```
$ docker exec pg bash -c 'psql --command "show work_mem;"'
work_mem
----------
64MB
(1 row)
```

更新了！

我们甚至可以更进一步，动态地改变这个值。让我们从更新我们的模板

```
work_mem = 64MB

```

到

```
work_mem = ${POSTGRESQL_WORK_MEM}

```

我们首先需要重建我们的形象

```
$ docker build --tag=tomastomecek/postgresql .
```

然后让我们通过环境变量将`work_mem`属性设置为 128MB:

```
$ docker run --rm \
    -e POSTGRESQL_USER=pg_test \
    -e POSTGRESQL_PASSWORD=secret \
    -e POSTGRESQL_DATABASE=test_db \
    -e POSTGRESQL_WORK_MEM=128MB \
    --name pg tomastomecek/postgresql
```

我们能够定义一个新的环境变量，因为容器的启动脚本正在使用`envsubst`命令。显然，这只是一个实现细节，应该清楚地记录如何定义新变量(如果可能的话)。

现在让我们验证一下`work_mem`是否设置为 128MB。

```
$ docker exec pg bash -c 'psql --command "show work_mem;"'
work_mem
----------
128MB
(1 row)
```

### 赞成的意见

*   便携——容器在任何环境下都会以相同的方式工作。
*   易于测试和审计。

### 骗局

*   构建和分发大量新图像可能会很复杂。
*   需要构建映像——这需要额外的自动化(dockerfile 的 git 存储库、构建管道、映像命名约定、注册表)。
*   没有文档很难搞清楚——这可能导致未定义的行为。

## 注入配置

在 OpenShift 中，我们可以利用 [ConfigMaps](https://docs.openshift.com/enterprise/3.2/dev_guide/configmaps.html) 并使用它们在 pod 中注入配置。克里斯托夫·格伦为[写了一篇关于 OpenShift 内部配置最佳实践的博文](https://blog.openshift.com/flexible-container-images/)。

让我们在不构建新映像的情况下，执行上面“覆盖配置”一节中的示例。

我们将在 RHSCL postgresql 容器中绑定安装模板，而不是构建新的映像。

```
$ docker run --rm \
    -e POSTGRESQL_USER=pg_test \
    -e POSTGRESQL_PASSWORD=secret \
    -e POSTGRESQL_DATABASE=test_db \
    -v openshift-custom-postgresql.conf.template:/usr/share/container-scripts/postgresql/ \
    -e POSTGRESQL_WORK_MEM=128MB \
    --name pg \
    registry.access.redhat.com/rhscl/postgresql-95-rhel7:latest
```

这简单多了。我们要做的就是:

1.  获取模板文件。
2.  更新一下。
3.  将模板文件挂载到容器中。

让我们验证一下`work_mem`属性是否设置正确。

```
$ docker exec pg bash -c 'psql --command "show work_mem;"'
work_mem
----------
128MB
(1 row)
```

### 赞成的意见

*   从不可变的映像中分离配置——您可以独立地配置容器化的服务。
*   不需要创建新的图像。

### 骗局

*   容器不再是可移植的，需要配置。

## 结论

正如您所看到的，对于容器化服务的配置，没有什么灵丹妙药。有多种选择，由你来选择最适合你的那个。

* * *

**点击此处了解** [**Red Hat Openshift 容器平台**](https://developers.redhat.com/products/openshift/overview/) **，允许您供应、管理和扩展基于容器的应用程序。**