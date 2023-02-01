# 使用 Java 构建您的“Hello World”容器

> 原文：<https://developers.redhat.com/articles/java-container>

在阅读了本系列的前一篇博客文章“Containers，Kubernetes，microservice:Start here”之后，您现在已经准备好构建您的第一个“Hello World”应用程序，并在容器中运行它。为此，我们将使用 Java 和 Spring Boot。

## 建啊，波曼，还是码头工人

您使用哪种方法来构建和运行您的容器取决于您的操作系统和工具选择。因为它更安全(它不需要 root 访问权限)，我将使用 Podman 来构建和运行我的容器，因为我知道所使用的命令与`docker`命令 100%兼容。事实上，你可以运行`alias docker=podman`，你不会知道有什么不同。因此，如果您不使用 podman，只需使用命令`docker`代替下面的每个`podman`。

## 建筑用零件

您需要将要运行的代码、配置/管理构建过程的文件以及工具(即 Podman)。

## 构建配置/管理

我们将创建一个名为“Dockerfile”的文件，其中包含构建映像所需的步骤和信息。构建过程分层次完成，起点通常是操作系统，或者更可能是操作系统和框架的组合。例如，您需要安装 maven 和 Java JDK。所以从这一点开始是很常见的。您可以从操作系统开始，然后在构建映像时将框架全部安装到映像中——这样做可能有很好的理由。您也可以这样做——从一个操作系统开始，添加一个框架——保存该映像，并将其用作其他映像的基础。是的，像任何 IT 技术一样，你可以把它变得简单或复杂。为了简单起见，我们将从 OS+框架组合开始。然后我们将在映像中复制我们的代码，然后运行`mvn clean package`来构建我们的应用程序。最后，当有人在容器中运行图像时，我们将为图像提供一个要执行的命令。下面这个文件，“Dockerfile”(我从我的同事 Doug Tidwell 的 [Coderland 无服务器项目](https://developers.redhat.com/coderland/serverless/)中摘抄的)，做了那些事情:`FROM maven:3.5-jdk-8-alpine as builder``WORKDIR /app``COPY pom.xml .``COPY src ./src``RUN mvn clean package -DskipTests``FROM openjdk:8-jre-alpine``COPY --from=builder /app/target/helloWorld-*.jar /helloWorld.jar``ENV PORT 3333``CMD ["java","-Dserver.port=${PORT}","-jar","/helloWorld.jar"]`

A line-by-line explanation is later in this article, but let's just build thing and run it; we can come back to the details.

## 让我们得到一些代码

1.  派生或克隆 https://github.com/donschenck/path-to-kubernetes.的 github 库
2.  移动到目录 src/java/helloworld。

## 让我们建造并运行

To build the image, run`podman build -t hello-world-java .`Note that a fast internet connection really pays off here, as layers (images or binary code) are being downloaded to your build machine. If an image already exists in your build machine cache, the download is bypassed.To run the image (again, we'll dive into this later), run`podman run -p 3333:3333 hello-world-java`Finally, open a second terminal window and run`curl http://localhost:3333`You should see "Hello World!" as the result.

## 周期

So that's the basic cycle:

1.  创建源代码
2.  创建 Dockerfile 文件
3.  建立形象
4.  在容器中运行图像

## 关于那个文件

文件“Dockerfile”用于指导您构建图像。下面是一个简短的逐步分解:

`maven:3.5-jdk-8-alpine as builder`This is the builder image, a temporary image that contains the bits we need to build our solution.`WORKDIR /app`This simply establishes a working directory inside your image.`COPY pom.xml .``COPY src ./src`Copies files we need to the image.`RUN mvn clean package -DskipTests`Builds the solution inside the image.`FROM openjdk:8-jre-alpine`This is your base image, the starting point. In this case, it’s the official openjdk image and uses Alpine Linux as its base. That means we don’t have to install any framework; it’s already included with this base image. In case you’re wondering, it’s 85MB on my Mac.

![](img/8fafc0fe52266b3c9bb8bcb425ee7617.png)

`COPY --from=builder /app/target/helloWorld-*.jar /helloWorld.jar`Copies the compiled bits into the image.`EXPOSE 3333`Exposes the application port, 3333, to the outside world.`CMD ["java","-Dserver.port=${PORT}","-jar","/helloWorld.jar"]`This is what runs when the image is started (i.e. `podman run` or `docker run`)

## 在容器中运行

运行`podman run -p 3333:3333 hello-world-java`命令启动容器中的映像。它的代码使用端口 3333，它被映射到本地端口 3333。请随意尝试。它将附加到您的命令行；也就是说，它在运行时会占用您的终端。您可以通过在命令中使用`--detach`选项来消除这种情况。在这种情况下，容器在后台运行。您可以通过运行`curl`命令或打开浏览器到`http://localhost:3333`来查看代码的结果。

## 把所有的东西装箱

因此，您知道拥有在 Linux 容器中运行 Java 代码所需的所有知识和工具。下一篇博文将扩展这方面的知识，以包括一个应用程序的多个实例和/或一个集群中的多个应用程序。

*Last updated: June 10, 2019*