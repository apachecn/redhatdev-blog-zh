# 如何使用源 S2I 部署过程运行 FIS 2.0 应用程序

> 原文：<https://developers.redhat.com/blog/2017/08/14/how-to-run-fis-2-0-application-using-source-s2i-deployment-procedure>

本文描述了如何使用 s2i 源工作流创建和部署 FIS 2.0 项目。它从头开始创建一个项目，使用 github repository 可以将他们的基于 FIS 2.0 camel 和 spring-boot 的项目部署到 Openshift 环境中。下面是序列中的步骤，应该遵循这些步骤来轻松部署应用程序。

*   首先，根据文档[https://access . red hat . com/documentation/en-us/red _ hat _ JBoss _ fuse/6.3/html-single/fuse _ integration _ services _ 2.0 _ for _ Openshift/](https://access.redhat.com/documentation/en-us/red_hat_jboss_fuse/6.3/html-single/fuse_integration_services_2.0_for_openshift/)建立一个包含 FIS 映像和模板的 open shift 环境。
*   在本地路径的某个地方创建一个目录 spring-boot。现在转到这个文件夹。

```
mkdir spring-boot

cd spring-boot
```

*   执行以下命令下载 spring-boot 原型:

```
mvn org.apache.maven.plugins:maven-archetype-plugin:2.4:generate   -DarchetypeCatalog=https://maven.repository.redhat.com/ga/io/fabric8/archetypes/archetypes-catalog/2.2.195.redhat-000004/archetypes-catalog-2.2.195.redhat-000004-archetype-catalog.xml   -DarchetypeGroupId=org.jboss.fuse.fis.archetypes   -DarchetypeArtifactId=spring-boot-camel-xml-archetype   -DarchetypeVersion=2.2.195.redhat-000004
```

*   在 github repo 中，创建一个新的存储库。假设我们创建了“测试增益”。复制存储库 url。
*   现在创建一个本地 git 存储库，并将远程 github url 添加到本地 git 存储库中。

```
git init

git add *

git commit -m "Added spring-boot project"

git remote add origin https://github.com/1984shekhar/testingAgain.git

git push origin master
```

*   在 Openshift 控制台上，单击“添加到项目”。你可以在 Openshift GUI 的顶部面板找到它。
*   在“浏览目录”中搜索 spring。选择 s2i-弹簧靴-驼色。这个快速入门是驼色和弹簧靴的简单例子。
*   一旦选定，我们可以设置以下参数。而其他参数可以保持原样。

```
Application Name: test-camel-spring-boot

Git Repository URL: https://github.com/1984shekhar/sourceS2I_fis_example.git

#Git Reference refers to brnach name or tag.

Git Reference: master

Application Version: 1.0
```

*   点击创建，它将重定向到一个不同的页面。
*   点击页面上的“转到概述”。
*   转到“应用程序-> POD”，您应该会看到一个 POD。

```
test-camel-spring-boot-1-build with status Running.
```

*   点击“测试-骆驼-弹簧-启动-1-构建”窗格。
*   转到日志选项卡。滚动底部，它将下载神器。
*   最后，在几分钟的日志中，您会发现:

```
Pushing [==================================================>] 142.2 MB

Pushing

Pushed

latest: digest: sha256:756fe8a1b7fe53c174824c48b56d8b28e6ba48dadb9b9d0d4164ae76abebc58a size: 9646

Push successful
```

*   点击页面上的“转到概述”。
*   我们会看到另一个豆荚:

```
test-camel-spring-boot-1-3k38z in Running phase.

test-camel-spring-boot-1-build in Completed phase.
```

*   Pod 测试-camel-spring-boot-1-3k38z 是一个实际的 pod。点击它，并前往日志标签。
*   我们应该会看到如下日志:

```
08:16:39.411 [Camel (camel) thread #0 - timer://foo] INFO simple-route - 477

08:16:49.411 [Camel (camel) thread #0 - timer://foo] INFO simple-route - 497

08:16:59.411 [Camel (camel) thread #0 - timer://foo] INFO simple-route - 289

08:17:09.411 [Camel (camel) thread #0 - timer://foo] INFO simple-route - 872

08:17:19.412 [Camel (camel) thread #0 - timer://foo] INFO simple-route - 401
```

*   使用 oc 客户端，我们可以使用以下命令进行验证:

```
[cpandey@cpandey karaf-camel]$ oc get pods

NAME  READY STATUS RESTARTS AGE

test-camel-spring-boot-1-3k38z 1/1 Running 0 3mtest-camel-spring-boot-1-build 0/1 Completed 0  13m
[cpandey@cpandey karaf-camel]$
```

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/docker-cheatsheet/) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: January 12, 2018*