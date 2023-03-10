# OpenShift 容器平台上基于微概要文件的微服务——第 1 部分

> 原文：<https://developers.redhat.com/blog/2017/08/25/a-microprofile-based-microservice-on-openshift-container-platform-part-1>

# 创建一个简单的基于微概要文件的微服务，并将其部署到 OpenShift 容器平台

Eclipse MicroProfile 是企业 Java 微服务的开源规范。这是一个由个人、供应商和组织组成的社区，他们在现代开发、架构和底层基础设施的背景下，协作并致力于创新的企业 Java 微服务模式，例如云环境中的健康检查、容错、指标和安全传播。它的第一个版本基于 3 个 Java EE JSR/libraries/API，但这并不意味着 Eclipse MicroProfile 所做的一切都是以 Java EE 为中心的，一些 API 规范可能最终只是 MicroProfile 的一部分，这取决于社区本身和 Java EE ¹ 的规范领导。例如，Eclipse MicroProfile 1.1 的新版本包括 Config API，它是非 Java-EE API。Eclipse MicroProfile 项目的目标之一是创新，因此它的发布时间表比标准主体更灵活。

Eclipse MicroProfile 是一个规范，Red Hat 使用 WildFly Swarm 实现它，wild fly Swarm 是作为 Red Hat OpenShift 应用程序运行时的一部分交付的运行时。

这是关于基于 MicroProfile 的微服务的 3 篇博文系列的第一篇。在第一篇文章中，我将介绍如何创建一个简单的基于 MicroProfile 的微服务，以及如何在 OpenShift 容器平台上部署和运行它。在第二篇博文中，我将介绍如何将基于 MicroProfile 的微服务与数据库(由示例数据创建和填充)相关联。在上一篇博文中，我将介绍如何通过基于 MicroProfile 的微服务使用 JBoss 数据网格(缓存数据库信息)。

当然，你可以从头开始编写自己的基于 MicroProfile 的微服务，但是对于这个教学练习，我将使用 WildFly Swarm 项目生成器来创建一个示例 MicroProfile 微服务。

## 先决条件

在我们开始练习之前，这里是我在我的环境中使用的先决条件:

*   运行 MAC OS Sierra(10 . 12 . 6)/16 GB RAM/2.5 GHz 英特尔 i7 处理器的 MacBook Pro
*   【15353】
*   OpenShift 主机:v3.5.5.24
*   Kubernetes 主版本:v1.5.2+43a9be4

注意:我首先安装 Docker 来设置我的环境，然后打开 Shift 容器平台。你可以在这里 找到一篇关于如何做到这一点的好博文 [。](https://blog.switchbit.io/openshift-cluster-up-with-docker-for-mac/)

## 步骤

### 确保所需组件启动并运行

首先，确保 Docker 已经启动并运行。同样，m 通过打开一个浏览器窗口并输入:来确保 OpenShift 已启动 ²

*https://127 . 0 . 0 . 1:8443*

浏览器地址栏中的。此时，以用户名“developer”和密码“password”登录。

![](img/684c0c09904482d4599edcea1a52f44f.png)

登录后，您应该会看到以下内容:

![](img/132fd3d3313ae1ebc86c53e6327a18bf.png)

“我的项目”是默认项目，我们将在这些博客系列中使用。至此，您已经在笔记本电脑上运行了一个工作的 OCP 环境。

### 生成微文件微服务并为 OpenShift 容器平台做好准备

去野蜂项目发电机[【http://wildfly-swarm.io/generator/】](http://wildfly-swarm.io/generator/)。

首先，确保在“依赖项”字段中输入“MicroProfile”。

![](img/91b1b7e20782121c0f69b63754b5953f.png)

当出现名为“micro profile-Implementation of micro profile . io”的下拉菜单时，点击它。这将把 MicroProfile 依赖项添加到您的项目中(一个标题为“MicroProfile”的绿色框将出现在“Selected dependencies”下)。

![](img/51bdb9719b8fed4a7674027c5b2d9793.png)

在“组 ID”字段中输入“com.mpexample ”,在“工件 ID”字段中输入“mpHelloWorld”。

![](img/b79fa572b6ea2674c9260ba5e4fa4cd1.png)

点击“生成项目”按钮。这将生成一个名为“mpHelloWorld.zip”的文件，当要求将其下载到您的本地驱动器时，单击“确定”按钮(确保在单击“确定”下载之前选择了“保存文件”选项)。

zip 文件的内容是:

![](img/a64bb8324e9c0aaad91b8eeef97e33dc.png)

将该 zip 文件移动到您选择的目录中并解压缩(在 Mac 上，您只需双击 zip 文件，它就会自动解压缩)。

在继续之前，我们需要做一些编辑。将目录切换到您的项目目录:

1.  *cd <您选择的 dir>/mphello world*
2.  编辑项目的 pom.xml 文件，并从 Swarm 项目的<版本>标签中删除字符串“-SNAPSHOT”。

下面是 pom.xml 的图片的一个片段:

![](img/a72776b0a9065a235b091d7245ae5eec.png)

下面是图片:

![](img/c8ff8c768c15efdce0561c7ed699e04f.png)

3.  编辑 pom.xml 并声明一个变量，该变量包含 fabric8 maven 插件的版本号，方法是在<属性>部分的末尾添加下面一行:

*<fabric 8 . maven . plugin . version>3 . 1 . 92</fabric 8 . maven . plugin . version>*

下面是图片前的*片段:*

![](img/6853603a88bdec7d69017f39b4a9b8a4.png)

这里是后的*图:*

![](img/709f7ba9c0a42f5bcc8cddb758c95c66.png)

4.  通过在 pom.xml 的<插件>部分添加以下代码行，将 fabric8 maven 插件添加为第一个插件:

<插件>
<groupId>io . fabric 8</groupId>
<artifactId>fabric 8-maven-plugin</artifactId>
<版本>$ { fabric 8 . maven . plugin . version } </版本>
<执行>
<执行>
<目标>
<目标>/目标>
</目标>
/包括>
</生成器>
</配置>
</插件>

下面是之前的图片:

![](img/865394208cd3a43669fa7c9741d37e07.png)

这里是后的*图:*

![](img/37a3dba93272ad509f5708b9255ae34a.png)

5.  最后，确保保存 pom.xml 文件。

### 为 OpenShift 容器平台构建和部署您的基于 MicroProfile 的微服务

至此，随着 OCP 的启动和运行，将目录切换到您下载的项目目录。打开终端窗口，输入:

*cd <自选目录>/mphello world*

然后，通过输入:登录到正在运行的 OpenShift

*oc 登录 https://127 . 0 . 0 . 1:8443-u developer-p developer*

下面是您应该看到的屏幕快照:

![](img/4c78e4de8f77e8d1d8992af0153fa91a.png)

现在，通过输入:将您的项目构建并部署到 OpenShift

*mvn clean fabric 8:build fabric 8:deploy-DskipTests*

由于构建和部署，许多行将向下滚动。下面是构建和部署开始时输出行的快照:

![](img/e4f6c9b2b960b12de6c3fcf8c78383ac.png)

这里是构建和部署中间的输出行的快照，Docker 映像就是在这里实际创建的:

![](img/37e91a30fbc3cd88ba77a92aeacefe75.png)

这里是构建和部署结束时输出行的快照:

![](img/8ddc97232bada8894655b3dac606023f.png)

### 验证您的部署，创建路线，并测试您的基于微配置文件的微服务

至此，您的项目已经在 OpenShift 中作为一个 pod 进行了部署和启动。

进入 OCP 登陆页面(成功登录“开发者”后的页面)，点击“我的项目”。您将看到一个 pod 正在运行您的应用程序“mphello world”:

![](img/e5553680486b86e3992844039d5ffef3.png)

您可以忽略关于您的容器没有运行状况检查的信息消息(在本练习中，我们不会涉及这一点)。在您关闭信息消息后，您将看到:

![](img/f94e735d260d52b53a8394c1b2031add.png)

现在，您的 pod 已经启动并运行在 OpenShift 容器平台上，您必须为它创建一个路由，以便您可以从浏览器窗口调用它。

点击图标旁边的“mphelloworld ”,该图标看起来像一个带有两个箭头的小圆圈。你会看到:

![](img/5c9e73a60ba429f7903352844e04a1cb.png)

点击“路线:”标签旁边的“创建路线”，接受所有默认值:

![](img/bd7aaa1e080dc454cf009a449527a7b3.png)

向下滚动并点击“创建”按钮:

![](img/57e3269f17fff7adb23ae7a21752f346.png)

这将为这个微服务创建一个路由，以便可以从 OpenShift 容器平台外部调用它:

![](img/1ea7a8be8012a58c7978f0fecc7ff1f2.png)

您可以在上面的“交通”部分看到创建的路线。从 OpenShift 容器平台外部调用服务的 URL 为“http://mphelloworld-my project . 192 . 168 . 1 . 5 . xip . io”。

要测试应用程序，打开浏览器窗口并输入:

*http://mphelloworld-myproject.192.168.1.5.xip.io/hello*

你应该会看到字符串“Hello from WildFly Swarm”，这是微服务作为 JSON 返回的内容，出现在浏览器窗口:

![](img/0c7d2370ea64b0b1df3b9f05167431b6.png)

本系列的第一篇文章到此结束。在这篇博文中，您了解了如何使用 WildFly Swarm generator 生成基于 MicroProfile 的微服务，并对其进行修改以在 OpenShift 容器平台上构建、部署和运行。最后，通过从 web 浏览器调用，您验证了基于微概要文件的服务运行良好。

在这篇博文的第 2 部分中，您将学习如何添加一个数据库，填充它，并将其与基于微概要文件的微服务相关联，您将修改该微服务以访问数据库内容。

 1–2017 年 8 月 17 日，甲骨文宣布考虑在 Java EE 8 发布后(计划于 2017 年底)将 Java EE 迁移至开源基金会。Java EE 的治理流程将适应新的开源基金会，从 MicroProfile 提交 API 将很可能遵循新的流程 

 2–如果您使用“oc 集群启动”启动 OCP，那么如果您回收集群，您将丢失所有工作。要在重启之间保持集群的状态，请使用以下命令启动它" oc cluster up-host-data-dir =/my data-use-existing-config "

* * *

**[**红帽 OpenShift 容器平台**](https://www.openshift.com/container-platform/) **可供下载。****

***Last updated: August 22, 2017***