# 在 Red Hat OpenShift 应用程序中使用 MySQL 数据库

> 原文：<https://developers.redhat.com/blog/2019/10/23/using-a-mysql-database-in-your-red-hat-openshift-application>

在 Red Hat OpenShift 中创建一个 MySQL 数据库对开发者来说是有用的，这一点毋庸置疑。但是，一旦数据库准备好了，有了表和数据，如何在应用程序中使用这些数据呢？使用红帽 OpenShift 是不是有什么特别的魔力？pod 名称可以改变的事实呢？本文将带您完成访问运行在您的 [OpenShift](https://developers.redhat.com/openshift/) 集群中的 MySQL 数据库的必要步骤。

## 有代码

这篇博文的代码可以在相关的 GitHub repo 上找到。

## 我们去弄个数据库

创建 OpenShift 集群之后，第一步是在 OpenShift 中创建一个 MySQL 数据库。[我之前的文章](https://developers.redhat.com/blog/2019/07/18/mysql-for-developers-in-red-hat-openshift/)描述了如何在 OpenShift 集群中创建一个短暂的 MySQL 数据库。出于本文的考虑，我们做了以下假设:

1.  集群名为 *mysql。*
2.  项目名称为 *mysqsl-test。*
3.  数据库名为 *sampledb。*

如果您更改这些值中的任何一个，您将需要根据需要更改命令和脚本。

## 数据库启动并运行

通过阅读上一篇文章，您现在将拥有一个在 OpenShift 中运行的数据库。在那篇文章中，我们将 MySQL 数据库实例命名为 mysql 。这很重要；如果您使用了不同的名称，请记下来，因为我们很快就会用到它。

## 服务

在这个演示中，我们将在 OpenShift 集群的 pods 中运行两个非常简单的微服务， *getCustomer* 和 *getCustomerSummaryList* 。这些服务的源代码存在于[GitHub repo](https://github.com/redhat-developer-demos/mysql-openshift-ephemeral)的一个目录中(如前所述)，因此我们将使用 OpenShift 灵活的源到映像(或 S2I)构建特性，而不是在本地机器上创建映像并将其推送到我们的集群。

S2I 特性允许您在 OpenShift 中引用 GitHub repo，并从源代码触发自动构建。OpenShift 将获取源代码，对其进行分析，并根据源代码的类型(例如 Node.js、Ruby 等)进行编译。).生成的图像将存储在 OpenShift 集群的内部注册表中。

您可以选择配置您的 GitHub repo，以便每当一个 pull 请求被批准时向您的 OpenShift 集群发布一个 webhook，从而触发 OpenShift 托管的映像的重建。

为了从源代码构建我们的微服务，我们将使用以下命令。

**注意:**您需要修改 MYSQL_USER 和 MYSQL_PASSWORD 值，以匹配创建数据库时提供的凭证(在上一篇文章中)。如果没有这些值，就需要添加一个对 MySQL 实例有适当权限的用户。

```
oc new-app https://github.com/redhat-developer-demos/mysql-openshift-ephemeral.git --context-dir=src/getCustomer --name getcustomer -e MYSQL_HOST=mysql -e MYSQL_DATABASE=sampledb -e MYSQL_USER=mysql_userid_goes_here -e MYSQL_PASSWORD=mysql_password_goes_here
```

```
oc new-app https://github.com/redhat-developer-demos/mysql-openshift-ephemeral.git --context-dir=src/getCustomerSummaryList --name getcustomersummarylist -e MYSQL_HOST=mysql -e MYSQL_DATABASE=sampledb -e MYSQL_USER=mysql_userid_goes_here -e MYSQL_PASSWORD=mysql_password_goes_here
```

这两个命令将在 OpenShift 集群中启动构建，这大约需要一分钟。如果你跑`oc get pods`，你可以看到他们在跑。当它们完成时，我们必须调用运行在 OpenShift 集群中的服务。从群集外部无法访问它们；这是故意的。我们还将向群集添加一个网站。该网站将使用这两种服务，我们将通过一个公共 IP 地址和 URL 向世界公开该网站。

## 网站

要创建网站，我们将使用不同的方法。我们不使用源代码，而是将一个 Linux 映像放入 OpenShift。以下命令将把映像拉入 OpenShift 集群的内部映像注册表并启动它。

```
oc new-app --name mvccustomer --docker-image=quay.io/donschenck/mvccustomer:latest -e GET_CUSTOMER_SUMMARY_LIST_URI="http://getcustomersummarylist:8080/customers" -e GET_CUSTOMER_URI="http://getcustomer:8080/customer"
```

注意，我们在 OpenShift 中提供了定义路径的环境变量，这些变量指向我们刚刚构建的两个服务。

还有一小步。我们需要向世界展示这个网站，这样就可以从我们的桌面浏览器浏览到它。我们这样做了，并使用以下命令在 OpenShift 中创建了一个“route ”:

```
oc expose service mvccustomer --insecure-skip-tls-verify=false
```

此时，您有两个微服务将从您的 MySQL 数据库中检索一个客户列表和一个客户，运行在 OpenShift 中——就像我们短暂的 MySQL 数据库一样。我们也有一个使用这两种服务的网站。通过运行以下命令，我们可以看到我们网站的 URL:

```
oc get routes
```

查看 URI，您可以看到涉及我的服务、我的项目、我的集群和我的领域的明显部分。URI 格式如下:

```
http://{service_name}-{project-name}.app.{cluster_name}.{domain_name}
```

还记得我在上面的“让我们得到一个数据库”一节中提到的，我们如何假设一些名字吗？好吧，如果你决定改变，你会注意到这里。

## 发现

这个想法，你可以提前知道 URI 将会是什么样子，这要归功于 Kubernetes 的“服务发现”功能因为您为服务指定了一个名称，所以 Kubernetes 将跟踪与其相关联的 pod，即使 pod 名称发生了变化(例如，一个 pod 被删除并被新的 pod 替换)。更好的是:当您扩展到多个 pod 时，您仍然只有一个 URI。Kubernetes 负责单元之间的负载平衡。

## 启动网站

只需将 URL 粘贴到您的浏览器中，然后开始点击。

![](img/b54f21680a2dd64fc78ed431b7098c8d.png)

## 下一步是什么？

下一步是在 Red Hat OpenShift 中创建一个永久的(即*而不是*短暂的)MySQL 实例。那是另一篇文章。

*Last updated: July 1, 2020*