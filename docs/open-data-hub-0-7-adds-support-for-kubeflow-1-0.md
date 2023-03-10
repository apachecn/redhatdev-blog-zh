# Open Data Hub 0.7 增加了对 Kubeflow 1.0 的支持

> 原文：<https://developers.redhat.com/blog/2020/08/13/open-data-hub-0-7-adds-support-for-kubeflow-1-0>

[开放数据中心](https://developers.redhat.com/search?t=Open+Data+Hub) (ODH)是在[红帽 OpenShift 4](https://developers.redhat.com/products/openshift/getting-started) 上构建人工智能即服务(AIaaS)平台的蓝图。Open Data Hub 的 0.7 版本包括支持在 OpenShift 上部署 [Kubeflow](https://developers.redhat.com/blog/2020/09/18/kubeflow-1-0-monitoring-and-enhanced-jupyterhub-builds-in-open-data-hub-0-8) 1.0，以及在 OpenShift 持续集成(CI)系统上增加组件测试。本文探讨了最近的更新。

## OpenShift 上的 Kubeflow 1.0

对于 Open Data Hub 的 0.7 版本，我们专注于为 [opendatahub-io/manifests](https://github.com/opendatahub-io/manifests) 的`v1.0-branch-openshift`分支中的 Kubeflow 1.0 部署提供更新和修复。该版本包含在每个组件的`openshift`覆盖中成功部署 OpenShift 所需的所有更新或修复，每个组件都经过了仔细的测试和验证。 [kfctl_openshift](https://github.com/opendatahub-io/manifests/blob/v1.0-branch-openshift/kfdef/kfctl_openshift.yaml) `kfdef`清单将所有这些覆盖合并成一个文件，您可以使用该文件在 openshift 上安装 Kubeflow 1.0 。

如果您有兴趣，以下是我们为解决在 OpenShift 上部署 Kubeflow 1.0 时出现的问题而应用的修复程序:

*   **Istio** :添加 Kubeflow 0.7 版本中的 OpenShift 覆盖图。
*   **Pytorch** :给 Pytorch 角色添加终结者。通过`ConfigMap`添加一个覆盖图来定制`initContainer`。
*   **Katib** :将缺失的终结器添加到分配的角色中。
*   **笔记本控制器**:将`notebook-controller`和`jupyter-web-app`组件更新到 1.0 版本。
*   **谢顿**:增加`seldon-core-operator`的内存限制，防止`OutOfMemory`出错。更改用于变异和验证 webhooks 的名称和端口。

## 结合开放式数据中心和 Kubeflow

我们仍然在测试混合 Kubeflow 和 ODH 组件的能力。我们计划在下一个 Open Data Hub 版本中提供对这一特性的全面支持。如果要测试 ODH 组件和 TensorFlow Training (TFJob)操作符的组合部署，可以部署 [opendatahub-mix](https://github.com/opendatahub-io/odh-manifests/blob/master/kfdef/kfctl_openshift_mix.yaml) `kfdef`清单。

## 所有组件均可进行持续集成测试

在之前的 Open Data Hub 版本中，我们成功集成了 OpenShift 持续集成(CI)系统，并为`odh-manifests`中可用的一些组件创建了基本的冒烟测试。对于这个版本，我们关注于为 ODH 部署的所有组件添加功能测试。对于每个新的“拉”请求，我们都会运行一个完整的 ODH 部署，其中包括所有可用的组件。在快速冒烟测试之后，我们运行一套功能测试来确认每个部署的组件都正常工作。

下面是我们为每个组件添加的测试的快速概要:

*   **人工智能库和塞尔顿**:集成测试，以验证人工智能库操作员创建了我们的`SeldonDeployment`模型，并且所有部署的模型 API 都是在线的。
*   **气流**:验证气流操作员可以成功部署`AirflowBase`和`AirflowCluster`自定义资源。
*   **Prometheus** :验证 Prometheus 应用程序是否成功运行，门户是否启动并运行。
*   **Grafana** :验证`GrafanaDashboard`和`GrafanaDataSource`定制资源部署成功。
*   **JupyterHub** :验证 JupyterHub 服务器和数据库是否在线。目前，我们正致力于包括将利用 JupyterHub 服务器和笔记本的完全用户界面(UI)自动化的测试。
*   **Spark Operator** :验证`SparkCluster`和`SparkApplication`定制资源部署成功，并生成特定 Spark 作业的预期输出。
*   **Argo** :验证所需的 pod 正在运行，并且示例“Hello，world”工作流成功运行。

## 很少更新到 1.2

在测试 ODH 0.7 版本时，我们注意到 Seldon 操作符中的[部署失败](https://github.com/SeldonIO/seldon-core/issues/2009)。当[命名空间](https://developers.redhat.com/blog/2020/06/26/migrating-a-namespace-scoped-operator-to-a-cluster-scoped-operator/) Seldon 操作符从版本 1.1 自动更新到 1.2 时，这个失败是由于陈旧的 webhooks 造成的。从 Seldon 版本 1.2 开始，现有的 webhooks 将在运营商升级期间自动更新。

## 结论

如果你想了解最新的开放数据中心更新，欢迎加入我们的 ODH [社区会议](https://gitlab.com/opendatahub/opendatahub-community/-/wikis/Open-Data-Hub-Community-Meeting-Agenda)和[邮件列表](https://lists.opendatahub.io/admin/lists/)。如果您对 ODH 部署有任何疑问，请通过 ODH [GitHub 问题](https://github.com/opendatahub-io/odh-manifests/issues)页面联系我们。

*Last updated: October 19, 2021*