# 用 fast-jar 构建更快的 Quarkus 应用程序

> 原文：<https://developers.redhat.com/blog/2021/04/08/build-even-faster-quarkus-applications-with-fast-jar>

[Quarkus](/products/quarkus/getting-started) 已经很快了，但是如果你能用超音速、亚原子的 [Java](/topics/enterprise-java) 框架让[的内循环开发](/blog/2021/02/11/enhancing-the-development-loop-with-quarkus-remote-development/)更快呢？Quarkus 1.5 引入了`fast-jar`，一种新的打包格式，支持更快的启动时间。从 Quarkus 1.12 开始，这个伟大的特性成为 Quarkus 应用程序的默认打包格式。本文向您介绍了`fast-jar`格式及其工作原理。

**注**:[第九次年度全球 Java 开发者生产力报告](https://www.jrebel.com/resources/java-developer-productivity-report-2021)发现更多的开发者正在用 [Quarkus](/products/quarkus/getting-started) 实现商业应用。Quarkus 支持快速启动和响应的实时编码，这使得开发人员可以将更多的精力放在业务逻辑实现上，而不是浪费时间在诸如重新编译和重新部署代码以及不断重启运行时环境等工作上。

## 自定义类加载器

为了理解`fast-jar`的秘密解决方案，你需要理解 Java 类加载器的目的以及当一个 Java 应用启动时它处理什么。Java 类加载器将 Java 类动态加载到 Java 运行时环境(JRE)中的 Java 虚拟机(JVM)中。Java 类加载器处理这些函数，因此 Java 运行时不需要知道文件和文件系统在哪里。不幸的是，对于具有更多依赖性的 Java 应用程序来说，类加载器加载得更慢。原因是类装入器通常在 Java 应用程序依赖项的数量上采用 O(n) Big O 符号。

Quarkus 的`fast-jar`格式解决了这个问题！当您使用`fast-jar`格式创建一个应用程序时，Quarkus 使用一个定制的`ClassLoader`，它在应用程序构建时就已经知道了整个类路径。`ClassLoader`索引构建时写入的类和资源的位置，在启动时读取该位置。

通过下面的步骤，您可以自己了解遗留 JAR 格式和新的`fast-jar`格式之间的区别。

## 步骤 1:创建两个带有多个扩展的 Quarkus 项目

首先，使用一个 Maven 插件来搭建一个基于 Quarkus 1.11.6.Final 的新项目，它使用遗留的 JAR 打包格式:

```
$ mvn io.quarkus:quarkus-maven-plugin:1.11.6.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-legacyjar-started \
    -DclassName="org.acme.getting.started.GreetingResource" \
    -Dextensions="infinispan-client,rest-client,openshift, resteasy-jackson" \
    -Dpath="/hello"

```

这个命令生成一个`getting-legacyjar-started`目录，将`infinispan-client`、`rest-client`、`openshift`和`resteasy-jackson`扩展下拉到新的 Quarkus 项目中。

接下来，基于 Quarkus 1.12.2.Final 创建另一个项目，使用新的`fast-jar`格式:

```
$ mvn io.quarkus:quarkus-maven-plugin:1.12.2.Final:create \
    -DprojectGroupId=org.acme \
    -DprojectArtifactId=getting-fastjar-started \
    -DclassName="org.acme.getting.started.GreetingResource" \
    -Dextensions="infinispan-client,rest-client,openshift, resteasy-jackson" \
    -Dpath="/hello"
```

这个命令生成一个包含相同扩展名的`getting-fastjar-started`目录，但是使用`fast-jar`打包格式。

## 步骤 2:打包应用程序以比较格式

接下来，我们将打包应用程序，比较 Quarkus 使用遗留和`fast-jar`打包格式生成类和资源的不同之处。首先使用以下 Maven 命令打包遗留 JAR 应用程序:

```
$ mvn package -f getting-legacyjar-started
```

输出应以`BUILD SUCCESS`结束。然后，runnable JAR 将被直接打包到目标目录中，在那里还会创建其他资源，如`lib`和`classes`:

```
$ ls -al getting-legacyjar-started/target            
...
drwxr-xr-x   5 danieloh  staff     160 Mar 15 00:31 classes
drwxr-xr-x  67 danieloh  staff    2144 Mar 15 08:35 lib
-rw-r--r--   1 danieloh  staff  249897 Mar 15 08:35 getting-legacyjar-started-1.0.0-SNAPSHOT-runner.jar
...

```

现在，打包`fast-jar`应用程序:

```
$ mvn package -f getting-fastjar-started
```

构建完成后，您将在目标目录中找到一个新的自包含的`quarkus-app`文件夹:

```
$ ls -al getting-fastjar-started/target/quarkus-app        
...

drwxr-xr-x  3 danieloh  staff   96 Mar 15 14:25 app
drwxr-xr-x  4 danieloh  staff  128 Mar 15 14:25 lib
drwxr-xr-x  4 danieloh  staff  128 Mar 15 14:25 quarkus
-rw-r--r--  1 danieloh  staff  621 Mar 15 14:26 quarkus-run.jar
...
```

## 步骤 3:比较应用程序的启动时间

现在，让我们用打包的 JAR 文件运行这两个应用程序，看看它们启动的速度有多快。首先运行`legacy-jar`应用程序:

```
$ java -jar getting-legacyjar-started/target/getting-legacyjar-started-1.0.0-SNAPSHOT-runner.jar
```

一旦应用程序启动，您将看到 1.276 秒的启动时间，如下所示(运行时间可能会因您的环境而有所不同):

```
INFO  [io.quarkus] (main) getting-legacyjar-started 1.0.0-SNAPSHOT on JVM (powered by Quarkus 1.11.6.Final) started in 1.276s. Listening on: http://0.0.0.0:8080

```

要做一个快速的健全性测试，您可以使用一个`curl`命令来访问 RESTful API。然后，您将看到以下输出:

```
$ curl http://localhost:8080/hello

Hello RESTEasy

```

通过按键盘上的 **CTRL + C** 来停止开发模式。然后，运行`fast-jar`应用程序:

```
$ java -jar getting-fastjar-started/target/quarkus-app/quarkus-run.jar

```

一旦应用程序启动，您将看到 0.909 秒的启动时间，如下所示(运行时间可能会因您的环境而有所不同):

```
INFO  [io.quarkus] (main) getting-fastjar-started 1.0.0-SNAPSHOT on JVM (powered by Quarkus 1.12.2.Final) started in 0.909s. Listening on: http://0.0.0.0:8080

```

新的`fast-jar`定制`ClassLoader`提供了比传统 JAR 应用程序快 360 毫秒的启动时间。当您访问端点( `/hello`)时，您将得到与遗留 JAR 应用程序相同的输出(`Hello RESTEasy`)。

`fast-jar`格式允许默认的`ClassLoader`保持最少数量的 jar 打开，以适应容器分层架构。它也不需要在整个类路径中查找已知目录中缺少的资源，比如`META-INF/services`。

## 结论

在本文中，您了解了为什么用`fast-jar`打包的应用程序比用 Quarkus 的遗留 JAR 格式打包的应用程序更快。我们还做了一个快速练习，以便您可以自己比较启动时间。

虽然我们没有研究这个选项，但是如果在开发模式下运行这两个应用程序，启动时间可能几乎相同，因为开发模式不使用 JAR 文件进行打包。这种增强是针对生产环境的。

对于有许多扩展和依赖的应用程序来说，`fast-jar`格式很有用，对于部署到像 [Red Hat OpenShift](/products/openshift/overview) 这样的[容器](/topics/containers)环境的应用程序来说，这种优势更大。访问 Quarkus 登录页面[开始您的下一次 Quarkus 之旅](/products/quarkus/getting-started)。

*Last updated: October 7, 2022*