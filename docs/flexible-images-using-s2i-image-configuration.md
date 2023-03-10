# 灵活的映像或使用 S2I 进行映像配置

> 原文：<https://developers.redhat.com/blog/2017/11/29/flexible-images-using-s2i-image-configuration>

容器映像通常带有预定义的工具或服务，进一步配置的可能性很小或有限。这让我们想到了如何提供包含合理的默认设置，同时又易于扩展的图像。更有趣的是，这既可以在一台 Linux 主机上实现，也可以在一个编排好的 OpenShift 环境中实现。

源到图像(Source-to-image，S2I)在三年前被引入，允许开发者通过简单地提供源代码作为输入来构建容器化的应用。那么，为什么我们不能用它来制作配置文件作为输入呢？我们当然可以！

## 创建可扩展映像

Maciej Szulik 在一篇文章中已经描述了创建 S2I 构建器映像，创建可扩展且足够灵活的映像以通过定制配置进行调整并没有太大的不同。因此，让我们把重点放在使图像可配置的关键点上。

### 必需的脚本

每个构建器映像必须提供的两个脚本是:**汇编**和**运行，**都包含在 s2i/bin/目录下。

#### **组装**

**assemble** 脚本定义了如何组装应用程序映像。

让我们看看官方软件集合 nginx S2I 构建器的图像，看看它的默认行为。当您打开汇编脚本(下面的片段)时，您会看到默认情况下， *nginx* 构建器映像会在提供的源代码内的 *nginx-cfg/* 和 *nginx-default-cfg/* 目录中查找，它希望在那里找到用于创建定制应用程序映像的配置文件。

```
if [ -d ./nginx-cfg ]; then
  echo "---> Copying nginx configuration files..."
  if [ "$(ls -A ./nginx-cfg/*.conf)" ]; then
    cp -v ./nginx-cfg/*.conf "${NGINX_CONFIGURATION_PATH}"
    rm -rf ./nginx-cfg
  fi
fi

if [ -d ./nginx-default-cfg ]; then
  echo "---> Copying nginx default server configuration files..."
  if [ "$(ls -A ./nginx-default-cfg/*.conf)" ]; then
    cp -v ./nginx-default-cfg/*.conf "${NGINX_DEFAULT_CONF_PATH}"
    rm -rf ./nginx-default-cfg
  fi
fi
```

#### **运行**

**运行**脚本负责运行应用程序容器。在 [nginx](https://github.com/sclorg/nginx-container/blob/master/1.12/s2i/bin/run) 案例中，一旦应用程序容器运行，nginx 服务器就在前台启动。

```
exec /usr/sbin/nginx -g "daemon off;"
```

#### **标签**

要告诉 s2i 脚本应该在哪里，您需要在 Dockerfile 文件中定义一个标签:

```
LABEL io.openshift.s2i.scripts-url="image:///usr/libexec/s2i"
```

或者，您也可以指定定制的*汇编*和*运行*脚本，方法是将它们与您的源文件/配置文件放在一起；构建器映像中的脚本将被覆盖。

### 可选脚本

由于 S2I 提供了一套相当复杂的功能，你应该总是提供关于用户如何使用你的图片的文档。测试配置(或应用程序)可能也会派上用场。

#### **用途**

当容器运行时，这个脚本输出关于如何使用图像的指令。

#### **测试/运行和测试/测试应用**

这些脚本将测试应用程序源代码和构建器映像的运行。

为了更容易地创建这些图像，您可以利用 [S2I 容器图像模板](https://github.com/container-images/s2i-container-image-template)。

## 使用自定义配置扩展映像

现在让我们看看如何在现实世界中使用这样的图像。

同样，我们将使用上面的 nginx 图像来演示如何调整它，以构建一个具有自定义配置的容器化应用程序。

使用 source-to-image 进行配置的一个优点是，它可以在任何独立的 Linux 平台上使用，也可以在一个编排好的 OpenShift 环境中使用。

在 Red Hat Enterprise Linux 上，运行以下命令非常简单:

```
$ s2i build https://github.com/sclorg/nginx-container.git --context-dir=1.12/test/test-app/ registry.access.redhat.com/rhscl/nginx-112-rhel7 nginx-sample-app
```

s2i build 命令获取 *test/test-app* /目录中的配置文件，并将它们注入到输出 *nginx-sample-app* 映像中。

注意，默认情况下，s2i 将存储库作为输入，并在其根目录中查找文件。在这种情况下，配置文件位于子目录中，因此指定了- context-dir。

```
$ docker run --rm -p 8080:8080 nginx-sample-app
```

运行容器会显示一个网站，通知你 Nginx 服务器正在工作。

类似地，在 OpenShift 中:

```
$ oc new-app registry.access.redhat.com/rhscl/nginx-112-rhel7~https://github.com/sclorg/nginx-container.git --context-dir=1.12/test/test-app/ --name nginx-test-app
```

*oc new-app* 命令创建并部署一个新的镜像 *nginx-test-app* ，使用 *test/test-app/* 目录中提供的配置进行修改。

创建路由后，您应该会看到与上面相同的消息。

## 使用 S2I 构建器映像进行扩展的优势

综上所述，有很多理由让你的项目利用 S2I。

*   **灵活性**——您可以通过提供一个配置文件来定制一个服务，以满足您的需求，这个配置文件可以重写容器中默认使用的值。这还没有结束:您是想安装一个默认情况下不包含在您的数据库映像中的附加插件，还是安装并运行任意命令？支持 S2I 的图像允许这样做。
*   **任何平台**——您可以使用 S2I 在几个提供 s2i RPM 包的 Linux 平台上构建独立的容器，包括 Red Hat Enterprise Linux、CentOS 和 Fedora，或者利用 s2i 二进制文件。S2I 构建策略是 OpenShift 中集成的构建策略之一，因此您也可以轻松地利用它来构建部署在编排环境中的容器。
*   **分离的图像和服务配置** -尽管图像和服务配置之间的清晰区分允许您执行更复杂的调整，但构建可再现性同时保持不变。

## 立即拉出并运行灵活的映像

下面的图片现在可以从 [Red Hat Container Catalog](https://access.redhat.com/containers) 的 S2I 构建器中获得，并且可以像上面演示的那样轻松扩展:

*   [MongoDB](https://access.redhat.com/containers?tab=tech-details#/registry.access.redhat.com/rhscl-beta/mongodb-34-rhel7)
*   [马里亚布](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/mariadb-102-rhel7)
*   [Nginx](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/nginx-112-rhel7)
*   [清漆](https://access.redhat.com/containers/?tab=overview#/registry.access.redhat.com/rhscl/varnish-4-rhel7)

更多的图像将很快出现在目录中。在此期间，你可以试试他们的[上游同行](https://github.com/sclorg)。

source-to-image 项目包含大量带有示例的文档，如果您想了解更多，请前往[项目的 GitHub 页面](https://github.com/openshift/source-to-image)。

## 资源

*   [https://github.com/openshift/source-to-image](https://github.com/openshift/source-to-image)
*   [http://docs . project atomic . io/container-best-practices/# _ builder _ images](http://docs.projectatomic.io/container-best-practices/#_builder_images)
*   [https://github . com/container-images/s2i-container-image-template](https://github.com/container-images/s2i-container-image-template)
*   [https://blog.openshift.com/override-s2i-builder-scripts/](https://blog.openshift.com/override-s2i-builder-scripts/)
*   [https://blog.openshift.com/create-s2i-builder-image/](https://blog.openshift.com/create-s2i-builder-image/)

* * *

[**加入红帽开发者计划**](https://developers.redhat.com/?intcmp=70160000000xZNgAAM) **(免费)并获得相关的备忘单、书籍和产品下载。**

*Last updated: September 3, 2019*