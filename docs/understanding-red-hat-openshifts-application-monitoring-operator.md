# 了解 Red Hat OpenShift 的应用程序监控操作符

> 原文：<https://developers.redhat.com/blog/2019/09/10/understanding-red-hat-openshifts-application-monitoring-operator>

监控系统通常由三层组成:托管指标数据的数据库层，在仪表板中以图形方式显示存储的指标数据的层，以及通过电子邮件、随叫随到通知系统和聊天平台等方法发送通知的警报层。本文概述了在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 的应用程序监控操作器中使用的组件，如何使用操作器安装它们，以及操作器的一个实例。

## 应用监控堆栈组件

Red Hat Integration 产品中使用的 OpenShift [应用程序监控操作符](https://github.com/integr8ly/application-monitoring-operator)旨在通过安装一组工具在集群上构建监控系统来覆盖所有这些层。这个监控栈由三个著名的开源社区组件组成，它们是通过部署 Grafana 和 Prometheus 操作符安装的: [Grafana](https://grafana.com) 、 [Prometheus](https://prometheus.io) 和 [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) 。

### 普罗米修斯

Prometheus 是一个开源项目，旨在监控微服务基础设施并提供警报。Prometheus 支持抓取应用程序实例来收集指标和生成图表。这些指标由服务通过 HTTP(S)公开，Prometheus 定期丢弃目标定义的端点，并将收集的样本写入其数据库。

这个工具附带了一个名为 [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) 的函数式查询语言，让用户可以实时选择和聚合时间序列数据。查询的结果可以显示为图形或表格数据，也可以通过 HTTP API 使用。

应用程序监控操作员使用 [Prometheus 操作员](https://coreos.com/blog/the-prometheus-operator.html)以简单、声明性的方式安装 Prometheus 实例，以在集群上创建、配置和管理 Prometheus 实例。

### Alertmanager

这里是处理从 Prometheus 服务器发送的警报的地方。Alertmanager 负责接收警报并将它们发送出去以通知最终用户。警报系统对于警告新问题是有用的，允许团队采取行动来防止重大问题或系统停机。

当警报发送到 Alertmanager 时，可以根据其严重性进行过滤和分组，然后系统决定采取何种措施，可以是保持沉默或通知接收者(电子邮件、聊天应用程序、呼叫等)。).

### 格拉夫纳

就监控系统而言，可视化层有助于团队在异常行为发生时进行检测并采取行动。在应用程序监控操作器中，我们使用 Grafana 作为可视化工具，以仪表板格式显示从 Prometheus 收集的时间序列数据。

Grafana 是使用 Grafana 操作员自定义资源(CR)安装的，它可以在集群中部署和管理 Grafana 实例。

## 应用程序监控操作员安装

在简要介绍了应用程序监控堆栈中的每个组件之后，让我们通过安装应用程序监控操作符来看看它们是如何协同工作的。我们首先需要的是 Red Hat OpenShift 集群，可以通过在本地机器上安装 Minishift 来提供。有关安装 Minishift 的更多信息，请参见[Minishift](https://docs.okd.io/latest/minishift/getting-started/index.html)入门。

一旦集群启动并运行，就可以开始部署操作器了:

1.  从 git 仓库克隆操作符:

    ```
    $ git clone git@github.com:integr8ly/application-monitoring-operator.git
    ```

2.  以管理员角色登录 OpenShift(需要有集群的管理员权限才能创建 CRDs、ClusterRoles 和 ClusterRoleBindings):

    ```
    $ oc login -u system:admin
    ```

3.  使用 Make 安装应用程序监视操作符，这将触发一个 shell 脚本在集群上创建一个新的名称空间，创建一个新的标签(`monitoring-key=middleware`)，并安装堆栈所需的所有自定义资源定义:

    ```
    $ make cluster/install
    ```

运行 Make 后，所有组件都被安装在一个名为`application-monitoring`的新名称空间中。为了确保安装顺利，您可以通过运行以下命令进行检查:

```
$ oc project
Using project "application-monitoring" on server "https://your-cluster-ip:8443"
$ oc get pods
```

![](img/9a382ae9f7493f8f1c7e8a99fad66282.png)

您应该能够通过获得 Prometheus、Grafana 和 Alertmanager 各自的路线来访问它们:

```
$ oc get route
```

![](img/620d14dabaa5a48a1ac41f482942cbc9.png)

要通过 UI 访问 Openshift，您可能需要向用户授予管理权限。为此，请运行以下命令:

```
$ oc adm policy add-cluster-role-to-user admin "user_name"
```

## 应用监控操作员在行动

既然应用程序监视操作符正在运行，那么让我们创建一个项目，向 Prometheus 公开指标。我们需要做的第一件事是在集群上创建一个新的名称空间来托管项目:

```
$ oc new-project example-prometheus-nodejs
```

应用程序监控操作员使用 [Kubernetes](http://developers.redhat.com/topics/kubernetes/) 标记系统来发现导入的定制资源(Prometheus 规则、服务监控器、Grafana 仪表板等)。):

*   *Prometheus 规则*在 Prometheus 中定义了一组警报规则。
*   *服务监控器*定义应该如何监控服务组，并且操作员根据定义自动配置 Prometheus 以废弃服务。
*   *Grafana 仪表板*定义要在 Grafana 中对账的仪表板。

可通过更改属性`labelSelector`在应用监控 CR 上更改标签值。要为集群上的新名称空间创建名为`middleware`的新标签，请运行:

```
$ oc label namespace example-prometheus-nodejs monitoring-key=middleware
```

在这个项目中，我们克隆的存储库是一个[模板文件](https://github.com/david-martin/example-prometheus-nodejs/blob/d647b83116519b650e00401f04c8868280c47778/template.yaml)，我们用它来部署应用程序、导入 CRs (Prometheus 规则、服务监视器、Grafana 仪表板等)。)并协调它们。

要部署应用程序，请使用以下命令:

```
$ oc process -f https://raw.githubusercontent.com/david-martin/example-prometheus-nodejs/master/template.yaml | oc create -f -
```

需要注意的一件重要事情是，Grafana 仪表板导入的自定义资源定义(CRD)没有 Grafana 检测和协调新仪表板所需的`monitoring-key=middleware`标签。要添加标签，请运行以下命令:

```
$ oc label grafanadashboard monitoring-key=middleware --all
```

在这个操作之后，我们应该会看到一个新的 Grafana UI 仪表板，显示应用程序的内存使用情况(以及其他情况)。你还应该在`example-prometheus-nodejs/example-prometheus-nodejs`中看到一个新的普罗米修斯目标，以及新的警戒角色`APIHighMedianResponseTime`。

### 额外资源

*   [普罗米修斯监控示例应用](https://github.com/david-martin/example-prometheus-nodejs/)
*   [普罗米修斯储存库](https://github.com/prometheus/prometheus)
*   [普罗米修斯文档](https://prometheus.io/docs/introduction/overview/)
*   [Grafana 文档](https://grafana.com/docs/)
*   [监控红帽整合](https://access.redhat.com/documentation/en-us/red_hat_integration/2019-07/html-single/monitoring_red_hat_integration/index)

*Last updated: July 1, 2020*