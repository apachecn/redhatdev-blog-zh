# 通过 OpenShift Do 在 Red Hat OpenShift 上使用自定义生成器图像

> 原文：<https://developers.redhat.com/blog/2019/07/15/using-a-custom-builder-image-on-red-hat-openshift-with-openshift-do>

使用 Red Hat OpenShift 我最喜欢的事情之一是开发者目录。开发者目录是一个中心位置，使用 Red Hat OpenShift 的团队可以在这里封装和共享应用程序组件和服务的部署方式。

开发人员目录通常用于定义一个基础设施模式，称为*构建器映像*。构建器映像是支持特定语言或框架的容器映像，遵循[最佳实践和源到映像( *s2i* )规范](https://docs.openshift.com/container-platform/4.1/openshift_images/create-images.html)。

OpenShift 开发人员目录提供了几个标准的构建器映像，支持用 Node.js、Ruby、Python 等语言编写的应用程序。虽然开发人员目录有许多简单的方法来开始部署几种受支持的语言，但该目录也非常灵活，允许您添加自己的构建器映像来支持目录中未预加载的基础架构模式。

## 自定义生成器图像

Golang 是目前目录中没有预装的一种语言。要添加 Golang 构建器图像，可以使用一个 *imagestream* 来定义使用什么容器图像来支持 Golang 组件。可以用 YAML 或 JSON 编写 imagestream 来定义构建器图像的属性，包括从哪里提取图像定义以及将在开发人员目录 web 控制台中显示的图像的描述。

我使用在这里找到的 imagestream 将 Golang 添加到 Red Hat OpenShift 4 开发人员目录中。我们感兴趣定义为 imagestream 一部分的第一个属性是 [*种类*属性](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L3)，它用于指定正在创建的 imagestream。我们还可以定义一个[显示名称](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L6)，它将通过 web 控制台出现在目录中。

imagestream 定义的主 [*标签*属性](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L11)可用于定义支持的构建器图像的不同版本。此外，tags 属性中定义的每个版本都可以有自己的标记，通过包含一个 [*构建器*标记](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L21)，可以指定该图像是一个构建器图像。

使用来自的*属性，我们可以指定使用什么图像以及从哪里提取图像。在 [Golang 示例](https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62#file-centos7-go-1-10-2-imagestream-L44)中，图像来自 DockerHub，是一个 Docker 图像。*

添加 Golang builder 映像的最后一步是使用`oc login`登录到您的 Red Hat OpenShift 集群，然后运行以下命令:

```
curl -kL https://gist.github.com/danielhelfand/6b63ba3f250ea596a077904d43abaf62/raw 2> /dev/null | oc apply -n openshift --as system:admin -f -
```

使用上面的命令，在 OpenShift 名称空间下添加了一个 Golang 选项，现在可以支持 Golang 应用程序。既然这个构建器映像在目录中可用，那么如何快速利用它来部署应用程序组件并测试该映像是否能够支持该组件呢？

使用[open shift Do(odo)](https://developers.redhat.com/blog/2019/05/03/announcing-odo-developer-focused-cli-for-red-hat-openshift/)(open shift 的面向开发人员的 CLI)，我们可以轻松利用开发人员目录中可用的构建器映像。为了查看哪些选项可用，odo 提供了一个命令来列出目录中可用的构建器图像(即组件):

```
$ odo catalog list components

NAME               PROJECT      TAGS
dotnet             openshift    1.0,1.1,2.1,2.2,latest
golang             openshift    1.10.2,latest
httpd              openshift    2.4,latest
java               openshift    8,latest
modern-webapp      openshift    10.x,latest
nginx              openshift    1.10,1.12,1.8,latest
nodejs             openshift    10,6,8,8-RHOAR,latest
perl               openshift    5.24,5.26,latest
php                openshift    7.0,7.1,7.2,latest
python             openshift    2.7,3.5,3.6,latest
ruby               openshift    2.3,2.4,2.5,latest
```

使用此处[所示的 OpenShift Golang 示例](https://github.com/sclorg/golang-ex)，我可以运行以下代码在我的 OpenShift 集群中创建一个项目，创建一个 Golang 组件，该组件将使用通过 imagestream 创建的 Golang builder 映像，通过 URL 公开该组件，然后部署该组件并让它在 OpenShift 上运行:

```
# Clone the golang example application and go into its root directory
$ git clone https://github.com/sclorg/golang-ex
$ cd golang-ex

# Create the configuration for the golang example application to be
# deployed to OpeShift using odo and deploy it to OpenShift

$ odo project create goproject
$ odo create golang golang-ex --port 8080
$ odo url create golang-ex --port 8080
$ odo push
```

部署完成后，可以通过 odo 创建的 URL 访问 Golang 组件。单击 URL 应显示以下问候语:

> >你好 OpenShift！

Imagestreams 使促进基础架构模式的重用变得更容易，因此开发团队可以专注于创建应用程序，而不是如何托管它们。odo 通过提供一个命令行界面进一步扩展了他们的能力，该界面使得在 OpenShift 上开发应用程序时可以简单地挑选和选择要使用的基础设施。

关于 imagestreams 的更多信息可以在 [Red Hat OpenShift 文档](https://docs.openshift.com/container-platform/4.1/openshift_images/images-understand.html#images-imagestream-use_images-understand)中找到。官方的 Red Hat 和社区支持的 imagestreams 可以在[open shift/library](https://github.com/openshift/library/tree/master/community)GitHub repository 以及[Software Collections](https://github.com/sclorg)GitHub 组织下找到。

关于 odo 的信息，你可以访问 [OpenShift Do](https://openshiftdo.org/) 网站，也可以查看该项目的 [GitHub 库](https://github.com/openshift/odo)。

*要看本教程的视频版本，[请访问 OpenShift 博客](https://blog.openshift.com/from-red-hat-developers-blog-using-a-custom-builder-image-on-red-hat-openshift-with-openshift-do/)。*

*Last updated: September 3, 2019*