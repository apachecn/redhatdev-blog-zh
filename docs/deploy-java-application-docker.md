# 如何用 Docker 部署 Java 应用程序

> 原文：<https://developers.redhat.com/blog/2017/11/17/deploy-java-application-docker>

docker 容器化环境是一个软件平台，它提供了操作系统级虚拟化的抽象。要了解更多你可以阅读这篇文章:[https://docs.docker.com/engine/docker-overview/](https://docs.docker.com/engine/docker-overview/)

您需要一个可用的基于容器的应用程序。查看文档，为您的操作系统找到正确的安装步骤:[https://docs.docker.com/engine/getstarted/step_one](https://docs.docker.com/engine/getstarted/step_one/)。

在这里，我将创建一个 java 应用程序，并使用一个基于容器的应用程序来运行它。该示例包括以下步骤:

1.  使用以下命令创建一个目录。

```
   $ mkdir  java-docker-application

```

2.创建一个 java 类并将它保存为目录中的 Sample.java 文件，

**Java-docker-application as Sample.java**。

**//sample . Java**T2


```
   class Sample{ 
   public static void main(String[] args)  { 
   System.out.println("This is java application by using docker ");
   } 
   }

```

3.创建 Dockerfile 文件

does 文件不包含任何文件扩展名。请简单地用 Dockerfile 文件名保存它。

**// Dockerfile**

```
   FROM java:8
   COPY . /var/www/java
   WORKDIR /var/www/java
   RUN javac Sample.java
   CMD ["java", "Sample"]

```

你的文件夹必须如下图所示。

![](img/c1db0ddbe33ab4ccdc412127ffa687b9.png)

4.建立码头工人形象

现在，按照下面的命令创建一个图像。为了创建映像，我们必须以 root 用户身份登录。在本例中，我们切换到了 root 用户。在下面的命令中， **java-application** 是图像的名称。我们可以为我们的码头形象取任何名字。

```
   $ docker build -t java-application .

```

输出如下所示:

```
  [root@xyz java-docker-app]# docker build -t java-application .
  Sending build context to Docker daemon 3.072kB
  Step 1/5 : FROM java:8
  ---> d23bdf5b1b1b
  Step 2/5 : COPY . /var/www/java
  ---> Using cache
  ---> c34d8257c2c6
  Step 3/5 : WORKDIR /var/www/java
  ---> Using cache
  ---> 1c620a43fe4e
  Step 4/5 : RUN javac Sample.java
  ---> Using cache
  ---> b0d9d44eead7
  Step 5/5 : CMD java Sample
  ---> Using cache
  ---> 474c196d8cc1
  Successfully built 474c196d8cc1
  Successfully tagged java-application:latest

```

在成功构建映像之后，我们现在准备在下一步运行 docker 映像。

5.运行 Docker 图像

现在我们将使用下面的 run 命令运行 docker。

```
  $ docker run java-application
```

上述命令的输出如下所示。

```
  [root@xyz java-docker-app]# docker run java-application
  This is java application
  by using Docker
```

现在，docker 映像运行成功。除此之外，您还可以使用其他命令。

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: October 18, 2018*