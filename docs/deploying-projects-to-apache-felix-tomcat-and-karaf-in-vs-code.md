# 用 VS 代码将项目部署到 Apache Felix、Tomcat 和 Karaf

> 原文：<https://developers.redhat.com/blog/2020/04/09/deploying-projects-to-apache-felix-tomcat-and-karaf-in-vs-code>

我们正在扩展对不同开发环境中的容器和服务器的工具支持。我们现有的 VS 代码扩展，[红帽服务器连接器](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-server-connector)，只为红帽服务器和运行时提供功能，如 [WildFly](https://wildfly.org/) 、 [Minishift](https://www.okd.io/minishift/) 、[红帽 JBoss 企业应用平台](https://developers.redhat.com/products/eap/overview) (JBoss EAP)、以及[红帽容器开发工具包](https://developers.redhat.com/products/cdk/overview)。在本文中，我们将介绍[Red Hat Community Server Connector](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-community-server-connector)，它是我们的 [Visual Studio Code (VS Code)扩展](https://developers.redhat.com/products/vscode-extensions/overview)的最新成员。

Community Server Connector 使部署、运行、调试和测试开放服务网关计划(OSGi)、Java EE 和 Jakarta EE 以及其他针对不同服务器和运行时的项目变得前所未有的简单。这个新的 VS 代码扩展允许您使用与服务器连接器中相同的用户界面(UI)和灵活性来控制 Apache Felix、Apache Karaf 和 Apache Tomcat。别担心，我们也会继续增强 Red Hat Server Connector。

本文提供了对 Red Hat 服务器连接器的一般介绍。有关更详细的介绍，请参见我的[视频演示](https://youtu.be/8JIcEzoPhlE)，其中包括 Apache Felix、Apache Karaf 和 Apache Tomcat 的用例。

[https://www.youtube.com/embed/6lBmlxe9uDs?autoplay=0&start=0&rel=0](https://www.youtube.com/embed/6lBmlxe9uDs?autoplay=0&start=0&rel=0)

## 一般特征

服务器连接器和社区服务器连接器共享一个公共 UI，该 UI 由[远程服务器协议(RSP) UI 扩展](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-rsp-ui)提供。安装任何一个服务器连接器都会自动引入这种依赖性。服务器连接器扩展可以启动和停止 RSP 服务器。与语言服务器协议(LSP)实例非常相似，RSP 允许您在使用服务器或运行时时执行日常任务。

远程服务器协议为下列主要任务提供支持:

*   在磁盘上找到服务器运行时，并对其进行配置以供使用。
*   将服务器运行时下载到磁盘，并对其进行配置以供使用。
*   以正常或调试模式启动服务器。(此时只能调试用 Java 写的服务器。)
*   停止服务器。
*   向服务器添加部署、部署它、取消部署它，等等。
*   运行服务器类型公开的任意操作。

## 支持 Apache 运行时

接下来，我们将看看 Red Hat Community Server Connector 对三个 Apache 运行时的支持:Apache Felix、Apache Karaf 和 Apache Tomcat。

### 阿帕奇菲利克斯

Community Server Connector 支持 [Apache Felix](https://felix.apache.org) 版本 3.2、4.6、5.6 和 6.0。Apache Felix 不像其他容器或 web 服务器那样功能齐全，所以我们的工具必然是有限的。当 Apache Felix 通过命令行启动时，它通常会将您直接转储到 shell 中。不幸的是，我们的扩展不能启动服务器的启动命令；后台的一个`rsp-server`执行 start 并将输出转发给我们的 VS 代码扩展。因此，没有好的方法允许用户与 OSGi shell 交互。

我们仍然允许您通过 UI 将部署发布到 OSGi `bundle/`文件夹，但是容器不会重新加载更改，直到您重新启动。为了弥补这一点，Apache Felix 服务器适配器包含一个动作来停止服务器，擦除`felix-cache`文件夹，并再次启动容器。有关此功能的更多信息，请参见 [Red Hat 社区服务器连接器视频演示](https://youtu.be/8JIcEzoPhlE)。

### 阿帕奇·卡拉夫

红帽社区服务器连接器支持 [Apache Karaf](https://karaf.apache.org) 版本 4.8。启动时，Apache Karaf(像 Apache Felix 一样)会自动将您启动到一个 shell 中。在这个 shell 中，您可以安装特性、控制 OSGi 包等等。与 Apache Felix 一样，我们不能在启动时在 shell 中提供用户交互。然而，阿帕奇 Karaf 比阿帕奇 Felix 功能更加全面。启动完成后，在 VS 代码中的终端中，用户操作通过 SSH 连接到 Apache Karaf 的 shell，这为您添加或删除特性、控制部署和修改配置提供了更大的灵活性。

视频演示展示了 Apache Karaf 的两个有趣的用例。第一个包括部署和控制一个简单的 OSGi 包。第二部分向您展示了如何配置在 Apache Camel 中实现路线所需的工具。

### 阿帕奇雄猫

最终支持的运行时是 [Apache Tomcat](http://tomcat.apache.org) ，支持 Tomcat 版本 5.5、6.0、7.0、8.0 和 8.5。这里的功能相当典型。您可以在运行或调试模式下启动和停止服务器，还可以选择展开部署，以及对 HTML 或 JavaServer Pages (JSP)等静态文件进行增量更改。视频演示展示了 Apache Tomcat 的打包(WAR)和分解部署。

## 结论

如果您每天都在使用 Apache 服务器和运行时，您可能会发现[Red Hat Community Server Connector](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-community-server-connector)是 VS 代码环境中一个有用的扩展。[观看视频演示](https://youtu.be/8JIcEzoPhlE)完整介绍我们新的 VS 代码扩展，然后看看你能用它做什么。

*Last updated: June 29, 2020*