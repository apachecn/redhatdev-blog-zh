# 在 OpenShift 4.2 上安装 Kubeflow v0.7

> 原文：<https://developers.redhat.com/blog/2020/02/10/installing-kubeflow-v0-7-on-openshift-4-2>

作为[开放数据中心](http://opendatahub.io/)项目的一部分，我们看到了 Kubeflow 项目的潜力和价值，因此我们致力于在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 上启用 Kubeflow。我们决定使用 Kubeflow 0.7，因为这是这项工作开始时发布的最新版本。这项工作包括添加新的安装脚本，提供所有必要的更改，比如允许服务帐户在 OpenShift 上运行。

Kubeflow 的安装仅限于以下组件:

*   中央仪表板
*   Jupyterhub
*   Katib
*   管道
*   Pytorch，TF-工作(培训)
*   很少上菜)
*   Istio

在不久的将来，所有新的修复和特性都将被提交到 Kubeflow 项目的上游。

## 先决条件

要在 OpenShift 上安装 Kubeflow，有一些关于平台和工具的先决条件。

### 平台

要运行此安装，需要 OpenShift 作为平台。你可以使用 OpenShift 4.2 或者[Red Hat code ready Containers](https://developers.redhat.com/products/codeready-containers/overview)(CRC)。如果您选择 OpenShift 4.2，您所需要的只是一个可用的 OpenShift 4.2 集群。或者，你可以试试 try.openshift.com 上的[集群。](https://try.openshift.com/)

如果选择 CodeReady 容器，则需要一个 CRC 生成的 OpenShift 集群。以下是推荐的规格:

*   16GB 内存
*   6 个 CPU
*   45GB 磁盘空间

最低规格是:

*   10GB 内存
*   6 个 CPU
*   30GB 磁盘空间(CRC 的默认值)

**注意**:在最低规格下，部署 Kubeflow 组件时，CRC OpenShift 集群可能会在大约 20 分钟内没有响应。

在 crc 集群上安装 Kubeflow 时，有一个额外的覆盖层(名为“CRC”)来启用`kfctl_openshift.yaml`中的元数据组件。默认情况下，此覆盖被注释掉。取消对覆盖的注释以启用它。

### 工具

安装/卸载 Kubeflow 需要安装工具`kfctl`。从 [GitHub](https://github.com/kubeflow/kubeflow/releases/) 下载工具。此安装需要版本 0.7.0。

## 启用 Istio 安装 Kubeflow

如前所述，我们添加了一个 KFDef 文件来专门在 OpenShift 上安装 Kubeflow，并包含了对不同组件的修复。要在 OpenShift 4.2 上安装 Kubeflow 0.7，请遵循以下步骤。假设此安装将在 OpenShift 4.2 集群上运行:

1.  克隆`opendatahub-manifest` fork repo，默认为分支`v0.7.0-branch-openshift`:

```
$ git clone https://github.com/opendatahub-io/manifests.git
$ cd manifests
```

2.  使用 OpenShift 配置文件和本地下载的清单进行安装，因为在编写本文时，我们遇到了这个 Kubeflow [bug](https://github.com/kubeflow/kubeflow/issues/4678) ，它不允许在构建过程中下载清单:

```
$ sed -i 's#uri: .*#uri: '$PWD'#' ./kfdef/kfctl_openshift.yaml
$ kfctl build --file=kfdef/kfctl_openshift.yaml
$ kfctl apply --file=./kfdef/kfctl_openshift.yaml
```

3.  验证您的安装:

```
$ oc get pods
```

4.  启动 Kubeflow 门户网站:

```
$ oc get routes -n istio-system istio-ingressgateway -o jsonpath='http://{.spec.host}/'
http://<istio ingress route>/
```

## 删除 Kubeflow 安装

要删除 Kubeflow 安装，请按照下列步骤操作:

```
$ kfctl delete --file=./kfdef/<kfctl file name>.yaml
$ rm -rf kfdef/kustomize/
$ oc delete mutatingwebhookconfigurations.admissionregistration.k8s.io --all
$ oc delete validatingwebhookconfigurations.admissionregistration.k8s.io --all
$ oc delete namespace istio-system
```

## Kubeflow 组件

为了能够在 OpenShift 4.2 上安装 Kubeflow 0.7，我们添加了一些功能和修复，以缓解我们遇到的安装问题。以下是组件列表以及更改和使用示例的描述。

### OpenShift KFDef

KFDef 是一种规范，旨在控制 Kubeflow 部署的供应和管理。该规范通常以 YAML 格式发布，并遵循 Kubernetes 中流行的定制资源模式来扩展平台。随着 Kubeflow Operator 的即将加入，KFDef 正在成为用于 Kubeflow 部署和生命周期管理的定制资源。

KFDef 是建立在 Kubernetes 原生配置管理系统 Kustomize(T1)之上的。为了将 Kubeflow 部署到 OpenShift，我们必须创建一个新的 KFDef YAML 文件，为 OpenShift 定制 Kubeflow 组件的部署清单。由于 Kustomize 是每个组件的配置管理层，因此有必要添加特定于 OpenShift 的 Kustomize 覆盖层(当选择覆盖层时，应用于默认资源清单集的补丁)。

看一下在上面的部署步骤中使用的 OpenShift 特定的 KFDef 文件，在 [opendatahub-io/manifests](https://github.com/opendatahub-io/manifests/blob/v0.7-branch-openshift/kfdef/kfctl_openshift.yaml) 存储库中。

### 中央仪表板

如果您使用名称空间`istio-system`中的路径`istio-ingressgateway`来访问 Kubeflow web UI，中央仪表板就可以开箱即用。

首次访问 web UI 时，系统会提示您创建一个 Kubeflow 用户名称空间。这是创建单个命名空间的一次性操作。如果您想让笔记本服务器、管道等的 Kubeflow 部署可以访问额外的名称空间。，您可以创建 Kubeflow [配置文件](https://www.kubeflow.org/docs/other-guides/multi-user-overview/#manual-profile-creation)。默认情况下，中央仪表板没有启用身份验证。

### Jupyter 控制器

我们使用三个 Jupyter 控制器定制:一个定制笔记本控制器、一个定制配置文件控制器和一个定制笔记本映像。让我们来看看每一个。

#### 定制笔记本控制器

我们使用定制的笔记本控制器来避免在生成笔记本时创建的有状态集中设置`fsGroup: 100`的默认行为。对于 OpenShift 中的服务帐户，该值需要特殊的安全上下文约束(SCC)。更复杂的是，SCC 需要被授予一个服务帐户，这个帐户是在创建概要文件时才创建的，所以这不能在安装过程中完成。

相关链接:

*   [相关上游问题](https://github.com/kubeflow/kubeflow/issues/4617)。
*   [该控制器映像的存储库](http://quay.io/kubeflow/notebook-controller:v0.7.0)。
*   [出处在这里](https://github.com/crobby/kubeflow/tree/openshift-fixes)。

#### 自定义轮廓控制器

我们使用定制的概要文件控制器来避免新创建的带有标签`istio-injection: enabled`的概要文件的默认行为。该标签导致容器试图启动一个`istio-init`容器，该容器又试图使用`iptables`，这在 OpenShift 4.x 中是不可用的。该 init 容器将失败并导致笔记本开始失败。

相关链接:

*   [相关上游问题](https://github.com/kubeflow/kubeflow/issues/4566)。
*   [控制器映像的存储库](https://quay.io/repository/kubeflow/profile-controller?tag=v0.7.0&tab=tags)。
*   [出处在这里](https://github.com/crobby/kubeflow/tree/openshift-fixes)。

#### 自定义笔记本图像

我们还添加了我们自己的[定制笔记本图像](http://quay.io/kubeflow/tf-notebook-image)，它已经预先填充在图像选择下拉列表中。这个映像在`/home/jovyan`目录中提供文件系统权限。它为[提供了这里描述的](https://github.com/kubeflow/kubeflow/pull/4537)功能。

### Katib

卡提卜遭遇了两个主要问题。第一个是作为一个没有特权的用户不能干净地运行( [#960](https://github.com/kubeflow/katib/pull/960) 、 [#962](https://github.com/kubeflow/katib/pull/962) 、 [#967](https://github.com/kubeflow/katib/pull/967) )。第二，它通过改变 pod 来破坏 pod 中生成的安全上下文( [#964](https://github.com/kubeflow/katib/pull/964) )。这两个问题都在上游 Katib 存储库中得到了修复，Katib 现在在 OpenShift 上运行没有问题。

第二个[问题](https://github.com/kubeflow/katib/pull/964)是一个在依赖变异 webhooks 的应用程序中常见的模式，其中部分变异是向正在部署的 pod 添加一个 sidecar 容器。如果新容器不具有初始化的安全上下文，则 pod 准入策略控制器将阻止其部署。我们在 [KFServing 组件](https://github.com/kubeflow/kfserving/pull/613)中也看到了同样的问题。

### 管道

为了让 Kubeflow 管道在 OpenShift 上工作，我们必须为 Argo 指定`k8sapi`执行器，因为 OpenShift 4.2 不包括 Docker 守护进程和 CLI。相反，它默认使用 [CRI-O](https://www.redhat.com/en/blog/red-hat-openshift-container-platform-4-now-defaults-cri-o-underlying-container-engine) 作为容器引擎。我们还必须将终结器添加到 OpenShift 的工作流权限中，以便能够设置所有者引用。

这种做法允许运行基于 YAML 的管道，这些管道符合 Argo 关于`k8sapi`管道执行的规范，特别是在将参数和工件保存在卷中的情况下(如`emtpyDir`)，而不是作为基本图像层一部分的路径(如`/tmp`)。这一特定要求导致所有示例 Kubeflow Python 管道出现错误。要测试您的管道，请使用本文中[提供的欺诈检测管道。](https://developers.redhat.com/blog/2019/12/16/ai-ml-pipelines-using-open-data-hub-and-kubeflow-on-red-hat-openshift/)

对于`minio`安装，我们还创建了一个服务帐户，并授予该帐户以`anyuid`身份运行的权限。

### 培养

对于培训，我们必须对两个应用程序进行更改:PyTorch 和 TensorFlow jobs (tf-jobs)。

#### PyTorch

对于 PyTorch，我们不必对组件进行任何更改。然而，我们确实需要对[中的一个例子的 Dockerfile 进行修改。我们必须通过执行以下操作向 docker 文件添加所需的文件夹和权限，以运行示例 MNIST 测试:](https://github.com/kubeflow/pytorch-operator/tree/master/examples/mnist)

1.  更改 Dockerfile 文件以包括以下内容:

```
FROM pytorch/pytorch:1.0-cuda10.0-cudnn7-runtime
RUN pip install tensorboardX==1.6.0
RUN chmod 777 /var
WORKDIR /var
ADD mnist.py /var
RUN mkdir /data
RUN chmod 777 /data
ENTRYPOINT ["python", "/var/mnist.py"]
```

2.  构建 docker 文件并将其推送到您的注册表中:

```
podman build -f Dockerfile -t <your registry name>/pytorch-dist-mnist-test:2.0 ./
podman push <your registry name>/pytorch-dist-mnist-test:2.0
```

3.  将注册表映像 URL 添加到安装 YAML 文件中。我们在没有 GPU 的情况下测试了这个设置，我们的文件如下:

```
apiVersion: "kubeflow.org/v1"
kind: "PyTorchJob"
metadata:
name: "pytorch-dist-mnist-gloo"
spec:
pytorchReplicaSpecs:
Master:
replicas: 1
restartPolicy: OnFailure
template:
spec:
containers:
- name: pytorch
image: <your registry name>/pytorch-dist-mnist-test:2.0<
args: ["--backend", "gloo"]
# Comment out the below resources to use the CPU.
resources: {}
Worker:
replicas: 1
restartPolicy: OnFailure
template:
spec:
containers:
- name: pytorch
image: <your registry name>/pytorch-dist-mnist-test:2.0
args: ["--backend", "gloo"]
# Comment out the below resources to use the CPU.
resources: {}
```

4.  通过运行命令创建 PyTorch 作业

```
oc create -f v1/<filename.yaml>
```

5.  检查工作和主 PyTorch pods 是否运行无误。

#### TF-作业

为了在 OpenShift 上进行 TF-jobs 培训，我们必须为 OpenShift 的`tf-job-operator` ClusterRole 添加`tfjob/finalizers`资源，以便能够设置所有者引用。按照以下步骤运行示例 MNIST 培训作业:

1.  运行:

```
$ git clone https://github.com/kubeflow/tf-operator
$ cd tf-operator/examples/v1/mnist_with_summaries
```

2.  创建如下所示的`PersisentVolumeClaim`(我们必须将集群的`acceddModes`改为`RedWriteOnce`):

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
name: tfevent-volume
namespace: kubeflow
labels:
 type: local
app: tfjob
spec:
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 10Gi
```

3.  运行:

```
$ oc apply -f tfevent-volume/<new pvc filename>.yaml
$ oc apply -f tf_job_mnist.yaml
$ oc describe tfjob mnist
Events:
Type    Reason                   Age   From      Message
----    ------                   ----  ----      -------
Normal  SuccessfulCreatePod      12m tf-operator Created pod: mnist-worker-0
Normal  SuccessfulCreateService  12m tf-operator Created service: mnist-worker-0
Normal  ExitedWithCode           11m tf-operator Pod: kubeflow.mnist-worker-0 exited with code 0
Normal  TFJobSucceeded           11m tf-operator TFJob mnist successfully completed.
```

### 服务

对于服务，我们不得不对其中一个应用程序进行修改:Seldon

#### 很少

为了让 Seldon 在 OpenShift 上工作，我们必须删除分配给引擎容器的“8888”UID 值，该容器是服务模型 pod 的一部分。该值规定每次为模型提供服务时，其引擎控制器容器 UID 被赋予值“8888”，但是该值不在 OpenShift 中允许的 UID 值范围内。

下面是一个欺诈检测模型示例，可以让您自己尝试一下这个问题:

1.  使用以下示例创建一个很少部署的 yaml 文件:

```
{ "apiVersion": "machinelearning.seldon.io/v1alpha2", "kind": "SeldonDeployment", "metadata": { "labels": { "app": "seldon" }, "name": "modelfull", "namespace": "kubeflow" }, "spec": { "annotations": { "project_name": "seldon", "deployment_version": "0.1" }, "name": "modelfull", "oauth_key": "oauth-key", "oauth_secret": "oauth-secret", "predictors": [ { "componentSpecs": [{ "spec": { "containers": [ { "image": "nakfour/modelfull", "imagePullPolicy": "Always", "name": "modelfull", "resources": { "requests": { "memory": "10Mi" } } } ], "terminationGracePeriodSeconds": 40 } }], "graph": { "children": [], "name": "modelfull", "endpoint": { "type" : "REST" }, "type": "MODEL" }, "name": "modelfull", "replicas": 1, "annotations": { "predictor_version" : "0.1" } } ] } }
```

2.  通过运行以下命令安装此配置:

```
$ oc create -f "filename.yaml"
```

3.  验证是否有包含名称`modelfull`的 pod 正在运行。
4.  验证是否存在包含名称`modelfull`的虚拟服务。
5.  使用下面的示例`curl`命令，从终端向模型发送一个*预测请求*:

```
curl -X POST -H 'Content-Type: application/json' -d '{"strData": "0.365194527642578,0.819750231339882,-0.5927999453145171,-0.619484351930421,-2.84752569239798,1.48432160780265,0.499518887687186,72.98"}' http://"Insert istio ingress domain name"/seldon/kubeflow/modelfull/api/v0.1/predictions
```

### Istio

安装 Kubeflow 0.7 提供的默认 Istio 需要添加到 Istio 入口网关服务和`anyuid`安全上下文的路由。这些附加功能允许 Istio 作为特权用户运行多个由 Istio 组件使用的服务帐户。

## 后续步骤

开放数据中心团队目前专注于多个后续步骤或任务:

*   解决本文档中已经讨论过的组件问题，如 Pipeline 和 Katib。
*   在 OpenShift 4.2 上集成 Kubeflow 0.7 和 Red Hat 服务网格。
*   向 Kubeflow 社区提出本文档中讨论的变更建议。
*   与 Kubeflow 社区合作，在 Kubeflow 网站上添加官方 OpenShift 平台文档作为支持平台。
*   为开放式数据中心和 Kubeflow 之间的紧密集成设计解决方案，包括运营商重新设计。

*Last updated: June 29, 2020*