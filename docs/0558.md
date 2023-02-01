# 为什么不将运营商的逻辑与特定的 Kubernetes 平台结合起来呢？

> 原文：<https://developers.redhat.com/blog/2020/01/22/why-not-couple-an-operators-logic-to-a-specific-kubernetes-platform>

您可能会发现自己处于这样一种情况，即您认为只有当您的操作符运行在特定的 Kubernetes 平台上时，逻辑实现才应该发生。因此，您可能想知道如何从运营商那里获得集群供应商。在本文中，我们将讨论为什么依赖供应商不是一个好主意。此外，我们将展示如何解决这种情况。

## **为什么不开发基于供应商的解决方案？**

让我们考虑这样一个场景，只有在集群中安装了 Operator life cycle Manager([OLM](https://github.com/operator-framework/operator-lifecycle-manager))的情况下，我们的操作员才会采取进一步的行动。那么，我们是否可以不检查集群是否是 OpenShift 平台，因为 OLM 是默认提供的？

不，我们不能。请注意，除了在 OpenShift 4 中安装和提供的 OLM 之外。通过默认安装，它也可能不总是真实的。然后，必须强调 OLM 也可以安装在其他供应商的产品上。

因此，这个例子很好地说明了依赖这些假设可能导致的问题。

## 那么，最好的方法是什么？

处理这种情况的最佳方法是寻找解决方案所需的确切 API 资源，而不是使用假设操作员是否在特定的 Kubernetes 平台上运行的资源。

在上面的例子中，我们要查看 OLM 的定制资源定义(API 资源)是否存在，这对于解决方案是必不可少的。

如果你想在项目运行于 OpenShift 和一个[入口](https://kubernetes.io/docs/concepts/services-networking/ingress)时创建一个[路由](https://docs.openshift.com/container-platform/4.2/networking/routes/route-configuration.html)资源(而不是，例如，只有当它运行于 Minikube 时)，那么我们应该检查特定的 [v1。路线](https://docs.okd.io/latest/rest_api/apis-route.openshift.io/v1.Route.html)是否安装。

## **如何实施这种方法**

让我们从创建发现客户端开始:

```
    // Get a config to talk to the apiserver
    cfg, err := config.GetConfig()
    if err != nil {
        log.Error(err, "")
        os.Exit(1)
    }

    // Create the discoveryClient
    discoveryClient, err := discovery.NewDiscoveryClientForConfig(cfg)
    if err != nil {
        log.Error(err, "Unable to create discovery client")
        os.Exit(1)
    }
```

**注意**:默认情况下，在所有用 [Operator-SDK](https://github.com/operator-framework/operator-sdk) 构建的项目中，`cfg *rest.Config`是在`main.go`中创建的。

现在，我们可以搜索所有 API 资源，如下所示:

```
    // Get a list of all API's on the cluster
    apiGroup, apiResourceList , err := discoveryClient.ServerGroupsAndResources()
    if err != nil {
        log.Error(err, "Unable to get Group and Resources")
        os.Exit(1)
    }
```

注意，通过使用`kubectl api-resources`命令，可以检查集群中的可用资源，如下所示:

```
$ kubectl api-resources
NAME                              SHORTNAMES   APIGROUP                       NAMESPACED   KIND
bindings                                                                      true         Binding
componentstatuses                 cs                                          false        ComponentStatus
configmaps                        cm                                          true         ConfigMap
endpoints                         ep                                          true         Endpoints
events                            ev                                          true         Event
limitranges                       limits                                      true         LimitRange
namespaces                        ns                                          false        Namespace
nodes                             no                                          false        Node
persistentvolumeclaims            pvc                                         true         PersistentVolumeClaim
persistentvolumes                 pv                                          false        PersistentVolume
pods                              po                                          true         Pod
podtemplates                                                                  true         PodTemplate
replicationcontrollers            rc                                          true         ReplicationController
resourcequotas                    quota                                       true         ResourceQuota
secrets                                                                       true         Secret
serviceaccounts                   sa                                          true         ServiceAccount
services                          svc                                         true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io   false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io   false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io           false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io         false        APIService
...
```

现在，我们可以检查并验证资源是否在群集上:

```
    // Looking for group.Name = "apiextensions.k8s.io"
    for i := 0; i < len(apiGroup); i++ {
        if  apiGroup[i].Name == name {
             // found the api
        }
    }
```

**注意:**也可以使用其他属性，比如`Group`、`Version`和`Kind`。欲了解更多信息，请查看 [APIGroup](https://godoc.org/k8s.io/apimachinery/pkg/apis/meta/v1#APIGroup) GoDoc。

可以使用`apiResourceList`的结果:

```
    // Looking for Kind = "PersistentVolume"
    for i := 0; i < len(apiResourceList); i++ {
        if  apiResourceList[i].Kind == kind {
            // found the Kind
        }
    }
```

**注意**:查看 [APIResourceList](https://godoc.org/k8s.io/apimachinery/pkg/apis/meta/v1#APIResourceList) GoDoc 查看其他选项。

## 如何获取集群版本信息

让我们以一个代码实现为例，它也可以通过使用 [DiscoveryClient](https://godoc.org/k8s.io/client-go/discovery#DiscoveryClient) 来获取集群版本:

```
// getClusterVersion will create and use an DiscoveryClient
// to return the cluster version.
// More info: https://godoc.org/k8s.io/client-go/discovery#DiscoveryClient
func getClusterVersion(cfg *rest.Config) string {
	discoveryClient, err := discovery.NewDiscoveryClientForConfig(cfg)
	if err != nil {
		log.Error(err, "Unable to create discovery client")
		os.Exit(1)
	}
	sv, err := discoveryClient.ServerVersion()
	if err != nil {
		log.Error(err, "Unable to get server version")
		os.Exit(1)
	}
	return sv.String()
}
```

请记住，这些想法可能是有益的，并允许您动态地和有计划地管理您的解决方案。请注意，它们对于处理特定场景中的问题也很有用。然后，在安装和配置过程之外使用。

此外，在结束之前，我要感谢@Joe Lanford 和@Jeff McCormick，他们合作并为本文提供了反馈和意见。

*Last updated: June 29, 2020*