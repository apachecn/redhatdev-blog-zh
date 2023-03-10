# 自定义 OpenShift 项目创建

> 原文：<https://developers.redhat.com/blog/2020/02/05/customizing-openshift-project-creation>

我最近参加了一个由 Red Hat 全球合作伙伴支持团队举办的关于高级 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 管理的优秀培训。培训中最有趣的内容之一是如何定制默认的项目创建。这篇文章解释了如何使用 OpenShift 的`projectRequestTemplate`为项目允许消耗的资源添加默认控制。

首先，简单介绍一下背景。OpenShift 项目与 Kubernetes 名称空间同义，用于隔离项目之间的对象。默认情况下，通过身份验证的用户可以创建项目和消耗资源，最高可达全局`ClusterResource`限制。作为一名集群管理员，您可能希望添加关于项目可以消耗的资源数量的新的默认限制。OpenShift 提供了一种机制，通过创建一个由 OpenShift 的项目配置资源中的`projectRequestTemplate`参数引用的模板来实现这种设置。

**注意:**你可以在[配置-项目-创建](https://docs.openshift.com/container-platform/4.2/applications/projects/configuring-project-creation.html)的官方文档中了解更多关于这个特性的信息。但是，如果您以前没有创建或修改过模板，那么缺省文档可能会有所欠缺。

此示例概述了如何获取项目创建模板架构，以及如何对其进行配置以设置默认项目限制和默认容器限制。对于我们的示例，项目限制如下所示:

*   最多使用 10 个吊舱。
*   将每个项目限制在六个 CPU。
*   将每个项目的内存限制在 16GiB。
*   将一个项目的请求设置为四个 CPU。
*   将项目请求设置为 8gb 内存。
*   设置 20GB 永久存储的请求。

我们示例中的容器限制如下所示:

*   将每个容器限制为一个 CPU。
*   将每个容器的内存限制在 1GiB。
*   将默认请求设置为 500 毫 CPU。
*   设置 500 兆内存的默认请求。

**注:**要了解更多关于限额和配额的信息，请阅读[每个项目的配额设置](https://docs.openshift.com/container-platform/4.2/applications/quotas/quotas-setting-per-project.html)。

在定制模板之前，我们需要获得一个模式。以拥有`cluster-admin`权限的用户身份运行以下命令:

```
$ oc adm create-bootstrap-project-template -o yaml > template.yaml
```

## 自定义模板

查看默认模板，您可以看到它是一个框架，仅包含特定的设置，如`NAME`、`DISPLAYNAME`、`DESCRIPTION`、`ADMIN_USER`和`REQUESTING_USER`，并建立了合理的基于角色的访问控制(RBACs):

```
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}
    creationTimestamp: null
    name: ${PROJECT_NAME}
  spec: {}
  status: {}
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    creationTimestamp: null
    name: admin
    namespace: ${PROJECT_NAME}
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: admin
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ${PROJECT_ADMIN_USER}
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER

```

定制这个模板的重要部分是在`objects`节下添加对象，如`ResourceQuota`和`LimitRange`。首先，像平常一样制作`ResourceQuota`和`LimitRange`:

```
- apiVersion: v1
  kind: "LimitRange"
  metadata:
    name: project-limits
  spec:
    limits:
      - type: "Container"
        default:
          cpu: "1" 
          memory: "1Gi" 
        defaultRequest:
          cpu: "500m" 
          memory: "500Mi"
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: project-quota
  spec:
    hard:
      pods: "10" 
      requests.cpu: "4" 
      requests.memory: 8Gi 
      limits.cpu: "6" 
      limits.memory: 16Gi
      requests.storage: "20G"

```

现在，在模板的`objects`部分下添加`ResourceQuota`和`LimitRange`对象。在本例中，我通过动态包含项目名称来修改每个对象的名称:

```
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  creationTimestamp: null
  name: project-request
objects:
- apiVersion: project.openshift.io/v1
  kind: Project
  metadata:
    annotations:
      openshift.io/description: ${PROJECT_DESCRIPTION}
      openshift.io/display-name: ${PROJECT_DISPLAYNAME}
      openshift.io/requester: ${PROJECT_REQUESTING_USER}
    creationTimestamp: null
    name: ${PROJECT_NAME}
  spec: {}
  status: {}
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    creationTimestamp: null
    name: admin
    namespace: ${PROJECT_NAME}
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: admin
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: ${PROJECT_ADMIN_USER}
- apiVersion: v1
  kind: "LimitRange"
  metadata:
    name: ${PROJECT_NAME}-limits
  spec:
    limits:
      - type: "Container"
        default:
          cpu: "1" 
          memory: "1Gi" 
        defaultRequest:
          cpu: "500m" 
          memory: "500Mi"
- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: ${PROJECT_NAME}-quota
  spec:
    hard:
      pods: "10" 
      requests.cpu: "4" 
      requests.memory: 8Gi 
      limits.cpu: "6" 
      limits.memory: 16Gi
      requests.storage: "20G"
parameters:
- name: PROJECT_NAME
- name: PROJECT_DISPLAYNAME
- name: PROJECT_DESCRIPTION
- name: PROJECT_ADMIN_USER
- name: PROJECT_REQUESTING_USER

```

下一步是将配置安装到`openshift-config`项目中:

```
$ oc create -f template.yaml -n openshift-config
```

之后，将模板与`config.openshift.io/v1`的项目资源中的`projectRequestTemplate`相关联。运行以下命令编辑配置:

```
$ oc edit project.config.openshift.io/cluster
```

在文本编辑器中，设置配置规范，在`projectRequestTemplate`下包含模板的名称。我们模板本身的名字是`project-request`。因此，在“规格”部分，我们将添加:

```
apiVersion: config.openshift.io/v1
kind: Project
metadata:
  ...
spec:
  projectRequestTemplate:
    name: project-request

```

## **确认模板工作**

最后一步是测试您的配置。请记住，项目创建模板仅适用于在模板安装并与`projectRequestTemplate`关联之后创建的项目。以前创建的项目将不会被修改:

```
$ oc new-project test
...
$ oc describe project test
Name:		test
Created:	40 seconds ago
...
Quota:
	Name:			test-quota
	Resource		Used	Hard
	--------		----	----
	limits.cpu		0	6
	limits.memory		0	16Gi
	pods			0	10
	requests.cpu		0	4
	requests.memory		0	8Gi
	requests.storage	0	20G
Resource limits:
	Name:		test-limits
	Type		Resource	Min	Max	Default Request	Default Limit	Max Limit/Request Ratio
	----		--------	---	---	---------------	-------------	-----------------------
	Container	cpu		-	-	500m		1		-
	Container	memory		-	-	500Mi		1Gi		-

```

上面的输出确认了配额和资源限制已经自动应用到项目中。

*Last updated: June 29, 2020*