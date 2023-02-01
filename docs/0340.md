# 使用新的 Operator SDK 开发 Kubernetes 运算符的 5 个技巧

> 原文：<https://developers.redhat.com/blog/2020/09/11/5-tips-for-developing-kubernetes-operators-with-the-new-operator-sdk>

[Kubernetes 运营商](https://developers.redhat.com/topics/kubernetes/operators/)本季风靡一时，名不虚传。运营商正从主要由技术基础设施专家使用发展成为更主流的， [Kubernetes-native](https://developers.redhat.com/blog/2020/04/08/why-kubernetes-native-instead-of-cloud-native/) 管理复杂应用的工具。今天，Kubernetes 操作者对于集群管理员和 ISV 提供商非常重要，对于内部开发的定制应用程序也是如此。它们为类似于云提供商所提供的标准化运营模式提供了基础。运营商也打开了在 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 上实现完全可移植的工作负载和服务的大门。

新的 Kubernetes[Operator Framework](https://operatorframework.io)是一个开源工具包，可以让您以有效、自动化和可扩展的方式管理 Kubernetes 操作员。运营商框架由三个组件组成:[运营商 SDK](https://sdk.operatorframework.io) 、[运营商生命周期管理器](https://github.com/operator-framework/operator-lifecycle-manager)和[运营商 Hub](https://operatorhub.io) 。在本文中，我将介绍使用[操作符 SDK](https://sdk.operatorframework.io) 的技巧和诀窍。Operator SDK 1.0.0 版本于 8 月中旬发布，因此现在是了解它的好时机。

**注**:运营商框架计划由 CoreOS 发起，Red Hat 在过去一年中一直在努力，并于 2020 年 7 月进入云计算原生计算基金会的孵化阶段。

![Operator SDK Tips and Tricks](img/0532311feea1fc1635a4222fd00c81b5.png)

## 探索 Kubernetes Operator SDK

我利用暑假的时间，探索了一下新的运营商 SDK 1.0.0 版本。为了我的实验，我开发了使用[头盔](https://sdk.operatorframework.io/docs/building-operators/helm/quickstart/)、 [Ansible](https://sdk.operatorframework.io/docs/building-operators/ansible/quickstart/) 和 [Go](https://sdk.operatorframework.io/docs/building-operators/golang/quickstart/) 的操作符，并将它们部署在 vanilla Kubernetes 和 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 上。这些语言是由 Operator SDK 提出的，它们提供了从简单到非常复杂的操作符的一系列功能。当然，你也可以使用其他技术来开发你的操作符，比如 [Python](https://developers.redhat.com/blog/category/python/) 或者 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 。我找到了很好的资源来指导我——即，Kubernetes 操作者的' [Hello，World '教程](https://developers.redhat.com/blog/2020/08/21/hello-world-tutorial-with-kubernetes-operators/)、[操作者最佳实践](https://github.com/operator-framework/community-operators/blob/master/docs/best-practices.md)和 [Kubernetes 操作者的 Go 最佳实践](https://www.openshift.com/blog/kubernetes-operators-best-practices)——但是我对 Go 或 Ansible 并不熟悉，所以我绞尽脑汁。我分享的这些建议都是我希望在开始之前就知道的事情。我希望他们也能帮助你。

**注意**:我们将使用的所有代码示例和资源都可以在本文的 [GitHub 资源库中找到](https://github.com/redhat-france-sa/openshift-by-example-operators)。

## 技巧 1:处理默认 CRD 值

每个 Kubernetes 操作符都有自己的自定义资源定义(CRD)，这是用于描述 Kubernetes 集群中高级资源规范的语法。从初次用户的角度来看，越简单的 CRD 越好；然而，有经验的用户会喜欢高级调整选项。处理所有定制资源实例的默认值对于保持事情的简单和可配置性是至关重要的，但是每个工具的做法都有所不同。

例如，假设我们想要部署一个由两个组件组成的应用程序:一个 web 应用程序和一个数据库。第一次使用的用户可以使用简单的自定义资源来部署它，如下所示:

```
apiVersion: redhat.com/v1beta1
kind: FruitsCatalog
metadata:
  name: fruitscatalog-sample
spec:
  appName: my-fruits-catalog

```

我们还需要副本数量、持久存储、入口等高级选项。

### 使用 Helm 自定义资源默认值

Helm chart 定义了一个用于处理自定义资源默认值的`values.yaml`文件。使用基于 Helm 的操作符 SDK，很容易向我们的示例文件添加一致的值:

```
# Default values for fruitscatalog.
appName: fruits-catalog-helm
webapp:
  replicaCount: 1
  image: quay.io/lbroudoux/fruits-catalog:latest
  [...]
mongodb:
  install: true
  image: centos/mongodb-34-centos7:latest
  persistent: true
  volumeSize: 2Gi
  [...]

```

### 用 Ansible 自定义资源默认值

基于 Ansible 的运算符 SDK 没有提供现成的方式来添加句柄自定义资源默认值。我发现的技巧要求您对您的操作符项目进行三处修改。

首先，创建一个用于处理默认值的`roles/fruitscatalog/default/main.yml`文件。注意 Ansible 对[蛇案](https://en.wikipedia.org/wiki/Snake_case)的用法，与通常用于自定义资源属性的[骆驼案](https://en.wikipedia.org/wiki/Camel_case)不同。例如，Ansible 将`replicaCount`转换为`replica_count`，因此您必须在操作符中使用这种形式:

```
---
# defaults file for fruitscatalog
name: fruits-catalog-ansible
webapp:
  replica_count: 1
  image: quay.io/lbroudoux/fruits-catalog:latest
  [...]
mongodb:
  install: true
  image: centos/mongodb-34-centos7:latest
  persistent: true
  volume_size: 2Gi
  [...]

```

一旦这个文件出现在您的角色中，Operator SDK 将使用它来初始化用户提供的自定义资源中缺少的部分。这种方法的局限性在于，SDK 只实现了一级合并。如果用户只将`webapp.replicaCount`放入自定义资源中，其他默认的子属性将不会合并到`webapp`变量中。基本上，你将不得不使用 Ansible 的`combine()`过滤器显式地处理合并过程。

因此，在角色的最开始，我们需要确保我们将拥有一个完整的资源，该资源基于用户提供的内容并与默认内容合并:

```
- name: Load default values from defaults/main.yml
  include_vars:
    file: ../defaults/main.yml
    name: default_cr

- name: Complete Custom Resource spec with default values
  set_fact:
    webapp_full: "{{ default_cr.webapp|combine(webapp, recursive=True) }}"
    mongodb_full: "{{ default_cr.mongodb|combine(mongodb, recursive=True) }}"

```

这里的诀窍是，SDK 初始化的`webapp`和`mongodb`变量不能写入；你将不得不重新创建像`webapp_full`这样的新变量，并在这个变量的基础上创建你的 Ansible 模板。令人高兴的是，当使用`make run`或`ansible-operator run`在本地运行您的 Kubernetes 操作符时，这种方法完全有效。

### 使用 Go 自定义资源默认值

基于围棋的运营商 SDK 也需要自己的方法。你可以在控制器中定义一个初始化方法(如 [Kubernetes 操作者最佳实践](https://www.openshift.com/blog/kubernetes-operators-best-practices)中所述)，但是我相信有更好的方法来处理它。

使用 Kubernetes `apiextensions.k8s.io/v1` API，现在可以直接在 CRD 中定义默认值。在 Helm 和 Ansible 中，你可以[手动完成 CRD 的 OpenAPI 部分](https://master.sdk.operatorframework.io/docs/building-operators/ansible/reference/advanced_options/#custom-resources-with-openapi-validation)。对于基于 Go 的操作符，可以在 Go 代码中使用`+kubebuilder`注释:

```
// WebAppSpec defines the desired state of WebApp
// +k8s:openapi-gen=true
type WebAppSpec struct {
    // +kubebuilder:default:=1
    ReplicaCount int32 `json:"replicaCount,omitempty"`
    // +kubebuilder:default:="quay.io/lbroudoux/fruits-catalog:latest"
    Image   string      `json:"image,omitempty"`
    [...]
}

```

要启用这个选项，您必须调整项目的`Makefile`来强制 SDK 生成`apiextensions.k8s.io/v1`清单:

```
CRD_OPTIONS ?= "crd:trivialVersions=true,crdVersions=v1"

```

在项目中运行`make manifests`命令会生成一个完整的 CRD，其中包含未来定制资源实例的默认值:

```
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.3.0
  creationTimestamp: null
  name: fruitscatalogs.redhat.com
spec:
  group: redhat.com
  names:
    kind: FruitsCatalog
    listKind: FruitsCatalogList
    plural: fruitscatalogs
    singular: fruitscatalog
  scope: Namespaced
  versions:
    - name: v1beta1
    schema:
      openAPIV3Schema:
        [...]
          [...]
          spec:
            description: FruitsCatalogSpec defines the desired state of FruitsCatalog
            properties:
              [...]
              webapp:
                description: WebAppSpec defines the desired state of WebApp
                properties:
                  image:
                    default: quay.io/lbroudoux/fruits-catalog:latest
                    type: string
                  replicaCount:
                    format: int32
                    default: 1
                    type: integer
                  [...]

```

这是非常整洁的。

## 技巧 2:让操作员为 OpenShift 做好准备

运营商 SDK 的一个好处是它从`operator-sdk init`或`operator-sdk create api`开始搭建你的项目的大部分。这个脚手架是你部署你的操作员到 OpenShift 所需要的，但是它不是全部。在我的实验中，我发现了一个缺失的部分，它与基于角色的访问控制(RBAC)权限有关。从本质上讲，应该认可操作员在没有对集群的完全访问权的情况下完成其工作。

当生成 Kubernetes 资源时，操作者应该尝试将自己注册为资源的所有者。这使得监视资源和实现终结器变得更加容易。通常，操作员可以包含一个引用已创建 CR 的`ownerReference`字段:

```
ownerReferences:
  - apiVersion: redhat.com/v1beta1
    blockOwnerDeletion: true
    controller: true
    kind: FruitsCatalog
    name: fruitscatalog-sample
    uid: c5d7e996-013f-40ca-bd19-14ba73728eaf

```

默认的脚手架在 vanilla Kubernetes 上运行良好。但是在 OpenShift 上，为了设置`ownerReference`块，操作者需要能够在自定义资源被创建之后设置它的终结器。所以现在您必须为您的操作员添加额外的权限，如下所述。

### 使用 Helm 和 Ansible 添加 RBAC 权限

使用 Helm 和基于 Ansible 的操作符，您可以在`config/rbac/role.yaml`文件中配置 RBAC 权限。您通常会添加如下内容:

```
- apiGroups:
  - redhat.com
  resources:
  - fruitscatalogs
  - fruitscatalogs/status
  - fruitscatalogs/finalizers 	# Missing line that is not added by the SDK
  verbs:
  - create
  - delete
  - get
  - list
  - patch
  - update
  - watch
```

### 使用 Go 添加 RBAC 权限

使用基于 Go 的操作符，您可以使用`+kubebuilder:rbac`注释将 RBAC 权限直接设置到控制器源代码中。只需在您的`Reconcile`函数注释中添加如下内容:

```
[...]
// +kubebuilder:rbac:groups=redhat.com,resources=fruitscatalogs/finalizers,verbs=get;create;update;patch;delete

// Reconcile the state rfor a FruitsCatalog object and makes changes based on the state read and what is in the FruitsCatalogSpec.
func (r *FruitsCatalogG1Reconciler) Reconcile(req ctrl.Request) (ctrl.Result, error) {
	[...]
}
```

**注意**:这些权限可能会在未来的版本中默认添加。请参见[拉请求#3779: Helm Operator:为已创建的 API](https://github.com/operator-framework/operator-sdk/pull/3779)添加终结器权限，以了解详细信息和跟踪。

## 技巧 3:发现您正在运行的集群

人们期望操作员具有适应性，这意味着他们必须能够根据环境改变他们的行动和他们管理的资源。一个简单的例子是在 OpenShift 上运行时使用路由功能而不是入口。要进行这种改变，Kubernetes 操作员应该能够发现它所部署的 Kubernetes 发行版以及集群上已经安装的任何扩展。目前，这种高级发现只能使用基于 Ansible 和 Go 的操作符来完成。

### Ansible 的高级发现

在基于 Ansible 的操作符中，我们使用一个`k8s`查找来请求出现在集群中的`api_groups`。然后，我们应该能够检测到我们正在 OpenShift 上运行，并仅在适当的时候创建一个`Route`:

```
- name: Get information about the cluster
  set_fact:
    api_groups: "{{ lookup('k8s', cluster_info='api_groups') }}"

[...]

- name: The Webapp Route is present if OpenShift
  when: "'route.openshift.io' in api_groups"
  k8s:
    state: present
    definition: "{{ lookup('template', 'webapp-route.yml') | from_yaml  }}"

```

### 使用 Go 进行高级发现

使用基于 Go 的操作符，这种类型的发现稍微复杂一些。在这种情况下，我们使用`discovery`包中的一个特定的`DiscoveryClient`。检索之后，您可以请求检索 API 组并检测您是否在 OpenShift 上:

```
import {
    [...]
    "k8s.io/client-go/discovery"
}

// getDiscoveryClient returns a discovery client for the current reconciler
func getDiscoveryClient(config *rest.Config) (*discovery.DiscoveryClient, error) {
    return discovery.NewDiscoveryClientForConfig(config)
}

// Reconcile the state for a FruitsCatalog object and makes changes based on the state read and what is in the FruitsCatalogSpec.
func (r *FruitsCatalogG1Reconciler) Reconcile(req ctrl.Request) (ctrl.Result, error) {
	[...]
    // The discovery package is used to discover APIs supported by a Kubernetes API server.
    config, err := ctrl.GetConfig()
    if err == nil andand config != nil {
        dclient, err := getDiscoveryClient(config)
        if err == nil andand dclient != nil {
            apiGroupList, err := dclient.ServerGroups()
            if err != nil {
                reqLogger.Info("Error while querying ServerGroups, assuming we're on Vanilla Kubernetes")
            } else {
                for i := 0; i < len(apiGroupList.Groups); i++ {
                    if strings.HasSuffix(apiGroupList.Groups[i].Name, ".openshift.io") {
                        isOpenShift = true
                        reqLogger.Info("We detected being on OpenShift! Wouhou!")
                        break
                    }
                }
            }
        } else {
            reqLogger.Info("Cannot retrieve a DiscoveryClient, assuming we're on Vanilla Kubernetes")
        }
    }
    [...]
}

```

您可以使用这种机制来检测其他已安装的操作符、`Ingress`类、存储能力等等。

## 技巧 4:在基于 Go 的操作符中使用扩展 API

这个技巧是专门针对基于 Go 的操作者的。由于 Ansible 和 Helm 将所有东西都视为 YAML，所以您可以自由地描述任何您需要的资源。当执行 OpenAPI v3 验证或通过准入挂钩时，它们的验证将只发生在集群 API 端。

Go 是一种强类型语言，当你处理复杂的操作符和数据结构时，这显然是一个优势。使用 Go，您可以依靠集成开发环境(IDE)等工具来帮助您完成代码完成和内联文档。因此，在将 Kubernetes 资源提交到集群之前，您可以对它们进行验证。然而，当您想用 API 扩展构建一些东西时，您必须将它们集成为 Go 依赖项，并在您自己的客户端运行时中注册它们。我会告诉你怎么做。

**注意**:虽然下面的讨论对于熟悉 Go 和 Kubernetes 的开发人员来说似乎是显而易见的，但是我的背景是 Java，我花了一些时间才弄明白。

### 将 API 扩展集成为 Go 依赖项

首先，您必须在项目根目录下的`go.mod`文件中包含新的 API 扩展依赖项。为此，Go 模块要么使用一个`Git`标签，要么使用分支名称(在后一种情况下，似乎是将分支名称翻译成最新的提交散列)。按照我之前的例子，如果我想为一个`Route`资源使用一个特定于 OpenShift 的数据结构，我必须添加以下内容:

```
require (
   [...]
   github.com/openshift/api v3.9.0+incompatible   // v3.9.0 is the last tag. New releases are managed as branches
   // github.com/openshift/client-go release-4.5  // As an example of integrating the OpenShift-specific client lib
)

```

下一步是将一个或多个包注册到受支持的运行时方案中。这允许您在标准的 Kubernetes Go 客户端中使用`Route` Go 对象。为此，您必须修改在项目根目录下生成的`main.go`文件。新增一个`import`，将方案注册到`init()`功能中；

```
import (
    [...]
    routev1 "github.com/openshift/api/route/v1"
)

func init() {
    utilruntime.Must(clientgoscheme.AddToScheme(scheme))
    utilruntime.Must(redhatv1beta1.AddToScheme(scheme))
    utilruntime.Must(routev1.AddToScheme(scheme))
    // +kubebuilder:scaffold:scheme
}

```

最后，在您的 Go `Reconcile()`函数或另一个操作符包中，您将能够以一种强类型的方式操纵`Route`结构，这有助于保持您的进度。然后，您可以使用控制器中的标准客户端创建该对象:

```
return androutev1.Route{
    [...]
     Spec: routev1.RouteSpec{
        To: routev1.RouteTargetReference{
            Name:   spec.AppName + "-webapp",
            Kind:   "Service",
            Weight: andweight,
        },
        Port: androutev1.RoutePort{
            TargetPort: intstr.IntOrString{
                Type:   intstr.String,
                StrVal: "http",
            },
        },
        TLS: androutev1.TLSConfig{
            Termination:                   routev1.TLSTerminationEdge,
            InsecureEdgeTerminationPolicy: routev1.InsecureEdgeTerminationPolicyNone,
        },
        WildcardPolicy: routev1.WildcardPolicyNone,
    },
}
```

## 技巧 5:调整操作员资源消耗

我的最后一个建议是*注意你的资源*——这句话有双重含义。

在 Kubernetes 操作符的特定上下文中，我的意思是自定义资源由操作符控制器监视，通常称为*操作数*。将操作员配置为监视相关资源非常重要。虽然有一些关于监视依赖资源的非常好的文档(参见文档中的[依赖监视器](https://master.sdk.operatorframework.io/docs/building-operators/ansible/reference/dependent-watches/)、控制器监视的[资源](https://master.sdk.operatorframework.io/docs/building-operators/golang/tutorial/#resources-watched-by-the-controller)，以及使用[谓词进行事件过滤](https://master.sdk.operatorframework.io/docs/building-operators/golang/references/event-filtering/#using-predicates))，但是现在没有必要深入研究这些。重要的是要知道，观看更多的软件资源会影响你的物理资源，即 CPU 和内存。

这是这句话的第二个意思:一旦你的运营商开始增长——这可能发生得非常快——你应该仔细考虑它消耗的资源。默认的请求和限制设置了较低的值，应根据您的需要进行调整。对于基于舵或 Ansible 的操作者来说尤其如此。

在您开始提升 CPU 和内存之前，请确保您关注了您的操作员应该管理的*并发协调*。简而言之:您的运营商应该同时管理多少客户资源？默认情况下，Operator SDK 将该值设置为运行 Operator 的节点上的核心数。但是，如果您正在监视许多资源，并且您有大的节点，那么这个设置可以充当所消耗资源的倍增因子。此外，如果您的 Kubernetes 操作符的作用域是一个特定的名称空间，那么您可能需要对名称空间中的一个或两个定制资源进行 16 次并发协调。

### 管理并发协调

您可以很容易地使用`—max-concurrent-reconciles`标志来设置最大并发协调数。新的 Operator SDK 项目布局利用了 [Kustomize](https://kubernetes.io/blog/2018/05/29/introducing-kustomize-template-free-configuration-customization-for-kubernetes/) ，因此您必须像这样更改`config/default/manager_auth_proxy_patch.yaml`:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: controller-manager
  namespace: system
spec:
  template:
    spec:
      containers:
      - name: kube-rbac-proxy
        [...]
      - name: manager
        args:
        - "--metrics-addr=127.0.0.1:8080"
        - "--enable-leader-election"
        - "--leader-election-id=fruits-catalog-operator"
        - "--max-concurrent-reconciles=4"
```

之后，您可以用通常的 Kubernetes 方式设置资源请求和限制。

## 包裹

在本文中，我分享了在使用最新发布的 Kubernetes Operator SDK 1.0.0 开发操作符时让我的生活变得轻松的五个技巧。我有很强的 Java 背景，但不是 Ansible 或 Go，我讨论的问题都让我绞尽脑汁几个小时。对于有经验的开发人员来说，这些提示可能是显而易见的，但是我希望它们能够节省其他开发人员的时间。

你呢？你和 Kubernetes 运营商合作有什么诀窍？

*Last updated: April 7, 2022*