# 使用 Flyway、Kubernetes 和 OpenShift 容器化 SQL DB 变化

> 原文：<https://developers.redhat.com/blog/2018/01/10/flyway-containerized-db-changes>

在 DevOps 项目中，你有时会被从单一世界继承的实践所困扰。 在之前的项目中，我们检查了如何简单地将 SQL 更新和更改应用到 [OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift?sc_cid=70160000000QiULAA0) 集群中的关系数据库管理系统(RDBMS)数据库。

Edson Yanaga 在其出色的免费书籍:迁移到微服务数据库:从关系型整体数据库到分布式数据中完美地描述了微数据库模式演变模式。YouTube 上的[也提供了这些模式的视频演示。](https://www.youtube.com/watch?v=tRteUhd9eAI)

在这个博客系列文章中，我们将展示一个简单的方法来在 OpenShift 上的持续集成和持续交付(CI/CD)管道中实现所描述的模式。该系列分为两部分:

*   这篇文章展示了如何在 OpenShift 上使用 Flyway、Dockerfiles 和 Kubernetes 来处理 SQL 更新自动化。
*   未来的帖子将展示应用程序迁移模式，包括使用 OpenShift Jenkins2 管道的数据库迁移阶段。

该方法使用 docker 容器、 [Flyway](https://flywaydb.org/) 和 [Kubernetes 对象](https://kubernetes.io/)来自动更新/修补运行在 OpenShift 上的微数据库上的 SQL。

## 创建你的微型数据库

为了简单起见，我们将依赖一个 [docker image](https://hub.docker.com/r/jbossdevguidebook/beosbank_posgres_db_europa/) ，它提供了一个简单的 [Postgres](https://en.wikipedia.org/wiki/PostgreSQL) 数据库，带有一个定制的预构建数据集，但是您可以构建一个定制的数据库服务来遵循这个演示。

数据库托管在 Openshift 上，我们假设您对 Openshift、Kubernetes 和 docker 容器有基本的了解。您可以使用以下指令安装一个简单的迷你移位/CDK 集群[:](https://developers.redhat.com/products/cdk/overview/)

运行 OpenShift/minishift 安装后，以管理员身份使用 oc CLI 命令进行连接:

```
$ oc login https://192.168.99.100:8443 -u developer -p developer $ oc new-project ocp-flyway-db-migration 
```

将 anyuid scc 授予默认服务帐户，以便运行 docker 映像

```
 $ oc adm policy add-scc-to-user anyuid -z default 
$ oc new-app --docker-image=jbossdevguidebook/beosbank_posgres_db_europa:latest --name=beosbank-posgres-db-europa
```

![](img/aef98b1934c0cf48dbee08bc09822506.png)

[![Openshift view: Beosbank pod](img/4dcdd5c7fb3bff545bcf839862e2a346.png "beosbank-db-pod")](/sites/default/files/blog/2017/12/beosbank-db-pod.png)

OpenShift 视图:Beosbank pod " >

确定数据库 pod 名称，并连接到数据库..然后，您可以浏览数据库内容:

```
$ oc get pods
 NAME                                 READY     STATUS    RESTARTS   AGE
 beosbank-posgres-db-europa-1-p16bx   1/1       Running   1          22h

```

```
$ oc rsh beosbank-posgres-db-europa-1-p16bx
# psql -U postgres

```

![](img/aff66a5133495705a2f4acdb49e6bf84.png)

![](img/f730dc5418b673b44a1955c619be0a5c.png)

既然 RDBMS 已经启动并运行，我们可能会问如何在数据库上执行自动 SQL 更新。

从单块流程中，我们有各种选项来完成它，包括带有 Flyway 运行时的 SQL 批处理。在下一节中，我们将首先看到如何将 Flyway 更新容器化，然后使用 Kubernetes 将其自动化。

## 用 Flyway 运行时容器化 SQL 更新

Flyway 流程容器化的目的是提供一个动态容器，该容器可以使用 Java 数据库连接(JDBC)协议连接到数据库容器，以便执行 SQL 更新。

从 DockerHub 你可以找到很多 Flyway 的自定义图片。以下 docker 文件可用于在 OpenShift 上下文中获取更合适的文件:

```
FROM alpine
MAINTAINER "Nono Elvadas"

ENV FLYWAY_VERSION=4.2.0

ENV FLYWAY_HOME=/opt/flyway/$FLYWAY_VERSION \
 FLYWAY_PKGS="https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/${FLYWAY_VERSION}/flyway-commandline-${FLYWAY_VERSION}.tar.gz"

LABEL com.redhat.component="flyway" \
 io.k8s.description="Platform for upgrading database using flyway" \
 io.k8s.display-name="DB Migration with flyway " \
 io.openshift.tags="builder,sql-upgrades,flyway,db,migration"

RUN apk add --update \
 openjdk8-jre \
 wget \
 bash

#Download flyway
RUN wget --no-check-certificate $FLYWAY_PKGS &&\
 mkdir -p $FLYWAY_HOME && \
 mkdir -p /var/flyway/data && \
 tar -xzf flyway-commandline-$FLYWAY_VERSION.tar.gz -C $FLYWAY_HOME --strip-components=1

VOLUME /var/flyway/data

ENTRYPOINT cp -f /var/flyway/data/*.sql $FLYWAY_HOME/sql/ && \
 $FLYWAY_HOME/flyway baseline migrate info -user=${DB_USER} -password=${DB_PASSWORD} -url=${DB_URL}
```

Dockerfile 安装 wget、bash 和一个 Java 运行时环境，然后下载特定版本的 Flyway 二进制文件。安装了 Flyway 二进制文件。在`/var/flyway/data`上创建一个卷来保存我们希望在数据库上执行的 SQL 文件。

默认情况下，Flyway 会检查`$FLYWAY_HOME/sql/`文件夹中的 SQL 文件。

我们首先将提供的所有 SQL 文件从数据卷复制到`$FLYWAY_HOME/sql/`，并启动一个迁移脚本。数据库 url 和凭证应该作为环境变量提供。

注意:最初的想法是告诉 Flyway 从卷中读取 SQL 文件，而不是将它们复制或移动到容器主目录中。然而，我们面临着这个配置的问题:
[(参见 github 上的 flyway 问题 1807)](https://github.com/flyway/flyway/issues/1807)。实际上，Flyway 引擎将递归读取卷，包括隐藏的子文件夹。有人请求进行增强，以自定义此行为，并防止 Flyway 读取卷安装文件夹中的元数据文件。

使用以下命令构建映像:

```
$ docker build -t --no-cache jbossdevguidebook/flyway:v1.0.4-rhdblog . 
```

```
 ... 

2018-01-07 13:48:43 (298 KB/s) - 'flyway-commandline-4.2.0.tar.gz' saved [13583481/13583481] 
---> 095befbd2450 
Removing intermediate container 8496d11bf4ae 
Step 8/9 : VOLUME /var/flyway/data
 ---> Running in d0e012ece342
 ---> 4b81dfff398b
Removing intermediate container d0e012ece342
Step 9/9 : ENTRYPOINT cp -f /var/flyway/data/\*.sql $FLYWAY_HOME/sql/ && $FLYWAY_HOME/flyway baseline migrate info -user=${DB_USER} -password=${DB_PASSWORD} -url=${DB_URL}
 ---> Running in ff2431eb1c26
 ---> 0a3721ff4863
Removing intermediate container ff2431eb1c26
Successfully built 0a3721ff4863
Successfully tagged jbossdevguidebook/flyway:v1.0.4-rhdblog
```

数据库客户机现在可以作为 docker 映像使用。在下一节中，我们将看到如何在 OpenShift 中组合 Kubernetes 对象来自动化这个数据库的 SQL 更新。

## 忽必烈在行动

Kubernetes 提供了各种部署对象和模式，我们可以依赖这些对象和模式从基于“jbossdevguidelines/flyway:v 1 . 0 . 4-rhd blog”映像创建的容器中应用动态 SQL 更新:

*   部署配置
*   职位
*   cron job/调度作业
*   InitContainer 边车

在下一节中，我们将说明如何使用单个 Kubernetes 作业对象来执行动态 SQL 更新。SQL 文件将通过卷和配置图提供给容器。

### 从提供的 SQL 文件创建配置图![](img/ce6284409946a0adf597493c7b530604.png)

```
$ cd ocp-flyway-db-migration/sql
$ oc create cm sql-configmap --from-file=.
configmap "sql-configmap" created
```

### 创建一个作业来更新数据库。

提供了工作规范。为了保持简单，我们不去深入定制的细节。文件可以在我的 github repo 上找到。

*   包括保存数据库用户和密码凭据的秘密
*   管理职务历史记录限制；根据所需的策略重新启动策略

```
$ oc create -f https://raw.githubusercontent.com/nelvadas/ocp-flyway-db-migration/master/beosbank-flyway-job.yaml
```

检查作业是在 OpenShift 中创建的:

```
$ oc get jobs
NAME DESIRED SUCCESSFUL AGE
beosbank-dbupdater-job 1 1 2d

```

检查豆荚。一旦创建了作业，它就会生成一个由新 pod 执行的作业实例。

```
$ oc get pods
NAME READY STATUS RESTARTS AGE
beosbank-dbupdater-job-wzk9q 0/1 Completed 0 2d
beosbank-posgres-db-europa-1-p16bx 1/1 Running 2 6d
```

根据日志，作业实例已成功完成，并且已应用迁移步骤。

```
$ oc logs beosbank-dbupdater-job-wzk9q
Flyway 4.2.0 by Boxfuse
Database: jdbc:postgresql://beosbank-posgres-db-europa/beosbank-europa (PostgreSQL 9.6)
Creating Metadata table: "public"."schema_version"
Successfully baselined schema with version: 1
Successfully validated 5 migrations (execution time 00:00.014s)
Current version of schema "public": 1
Migrating schema "public" to version 1.1 - UpdateCountry
Migrating schema "public" to version 2.2 - UpdateCountry2
Migrating schema "public" to version 2.3 - UpdateZip
Migrating schema "public" to version 3.0 - UpdateStreet
Successfully applied 4 migrations to schema "public" (execution time 00:00.046s).
+---------+-----------------------+---------------------+---------+
| Version | Description | Installed on | State |
+---------+-----------------------+---------------------+---------+
| 1 | << Flyway Baseline >> | 2018-01-05 04:35:16 | Baselin |
| 1.1 | UpdateCountry  | 2018-01-05 04:35:16 | Success |
| 2.2 | UpdateCountry2 | 2018-01-05 04:35:16 | Success |
| 2.3 | UpdateZip      | 2018-01-05 04:35:16 | Success |
| 3.0 | UpdateStreet   | 2018-01-05 04:35:16 | Success |
+---------+-----------------------+---------------------+---------+
```

### 检查更新的数据库

```
$ oc rsh beosbank-posgres-db-europa-1-p16bx
```

```
# psql -U postgres
psql (9.6.2)
Type "help" for help.

postgres=# \connect beosbank-europa
beosbank-europa=# select * from eu_customer;
 id | city      | country | street             | zip | birthdate |firstname | lastname
----+-------------+------------------+-------------------+--------+------------+
 1 | Berlin     | Germany  | brand burgStrasse | 10115 | 1985-06-20 |Yanick | Modjo
 2 | Bologna    | Italy    | place Venice      | 40100 | 1984-11-21 |Mirabeau | Luc
 3 | Paris      | France   | Bld DeGaule       | 75001 | 2000-02-07 |Noe | Nono
 4 | Chatillon  | France   | Avenue JFK        | 55 | 1984-02-19 |Landry | Kouam
 5 | Douala     | Cameroon | bld Liberte       | 1020 | 1996-04-21 |Ghislain | Kamga
 6 | Yaounde    | Cameroon | Hypodrome         | 1400 | 1983-11-18 |Nathan | Brice
 7 | Bruxelles  | Belgium  | rue Van Gogh      | 1000 | 1980-09-06 |Yohan | Pieter
 9 | Bamako     | Mali     | Rue Modibo Keita  | 30 | 1979-05-17 |Mohamed | Diallo
 10 | Cracovie  | Pologne  | Avenue Vienne     | 434 | 1983-05-17 |Souleymann | Njifenjou
 11 | Chennai   | Red Hat Training | Gandhi street | 600001 | 1990-02-13 |Anusha | Mandalapu
 12 | Sao Polo  | Open Source | samba bld | 75020 | 1994-02-13 |Adriana | Pinto
 8 | Farnborough | UK | 200 Fowler Avenue | 208 | 1990-01-01 |John | Doe
(12 rows)

beosbank-europa=#
```

如果使用相同的迁移脚本重新运行批处理，由于数据库已经知道这些修改，日志中将显示一条警告，并且不执行任何更新:

```
Current version of schema "public": 3.0
Schema "public" is up to date. No migration necessary.
```

文章到此结束。希望你能学到一些对你的集装箱之旅有帮助的东西。

完整的源代码可以从我的 Github 库获得:
[https://github.com/nelvadas/ocp-flyway-db-migration](https://github.com/nelvadas/ocp-flyway-db-migration)

*Last updated: January 5, 2022*