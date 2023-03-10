# 通过使用 Operator SDK 开始使用 Golang 运算符

> 原文：<https://developers.redhat.com/blog/2019/10/04/getting-started-with-golang-operators-by-using-operator-sdk>

开源的 [Operator Framework](https://coreos.com/blog/introducing-operator-framework) 是一个管理 Kubernetes 本地应用的工具包。该框架及其功能提供了开发解决方案以简化一些复杂性的能力，例如在 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 上安装、配置、管理和打包应用程序的过程。它提供了使用客户端执行 CRUD 操作的能力，即在这些平台上创建、读取、更新和删除数据的操作。

通过使用操作符，不仅可以提供所有预期的资源，还可以在执行时动态地、编程地管理它们。为了说明这个想法，想象一下如果有人不小心更改了一个配置或者错误地删除了一个资源；在这种情况下，操作员可以在没有任何人工干预的情况下修复它。在本文中，我们将看看操作符和操作符 SDK。

**注意:**作为本内容的先决条件，必须遵循[入门](https://github.com/operator-framework/getting-started)指南中概述的步骤。

## 蜜蜂

在跟随[入门](https://github.com/operator-framework/getting-started)之后，首先要做的一件事就是运行命令`operator-sdk add api --api-version=cache.example.com/v1alpha1 --kind=Memcached.`该命令的目的是为 Memcached 类生成定制资源(CR)和定制资源定义(CRD)资源。这个命令正在创建具有组`cache.example.com`和版本`v1alpha1`T6 的 API，它唯一地标识了新的`Memcached`类型的 CRD。

因此，通过使用 Operator SDK 工具，我们可以创建我们的 API 和对象，在这些平台上代表我们的解决方案。[入门](https://github.com/operator-framework/getting-started)教程只添加单一种类的资源；然而，它可以根据需要有多种类型(1…N)。基本上，CRD 是我们定制对象的定义，CRs 是它的一个实例。

## 项目

管理器负责管理控制器，然后通过控制器，我们可以在集群端执行操作。为了更好地理解它是如何工作的，在这个例子中，其中一个步骤是用命令`$operator-sdk build user/image:tag`创建一个 Docker 映像，然后替换文件`operator.yaml`文件中的值`REPLACE_IMAGE`。这个文件描述了它构建的项目实例。注意，通过运行命令`kubectl create -f deploy/operator.yaml`,我们正在创建一个带有该图像的 pod。

## **展示**这个想法

让我们考虑一个经典场景，目标是让一个应用程序及其数据库运行在 Kubernetes 平台上。然后，一个对象可以代表应用程序，另一个对象可以代表数据库。通过用一个 CRD 来描述应用程序，用另一个来描述数据库，我们不会损害封装、单一责任原则和内聚性等概念。破坏这些概念可能会导致意想不到的副作用，比如难以扩展、重用或维护，等等。

总之，应用程序 CRD 将作为其控制器的数据库 CRD。想象一下，应用程序运行需要一个部署和服务，因此在本例中应用程序的控制器将提供这些资源。类似地，DB 的控制器将拥有其对象的业务逻辑实现。

这样，对于每个 CRD，应该根据[控制器-运行时间](https://github.com/kubernetes-sigs/controller-runtime)设置的设计生产一个控制器。

[![](img/aaf22f3fc56ac3ac685ef5f512d9aecf.png)](https://che.openshift.io/f?url=https://raw.githubusercontent.com/redhat-developer/devfile/master/getting-started/go/devfile.yaml/?sc_cid=7013a000002D1quAAC)

## 控制器主要功能

### 协调()

协调功能负责根据在资源上实现的业务逻辑来同步资源及其规范。这样，它就像一个循环一样工作，直到所有条件都匹配它的实现，它才会停止。下面是伪代码和一个阐明它的例子。

```
reconcile App {

   // Check if a Deployment for the app exists, if not create one
   // If has an error, then go to the beginning of the reconcile
 if err != nil {
       return reconcile.Result{}, err 
   } 

   // Check if a Service for the app exists, if not create one 
   // If has an error, then go to the beginning of the reconcile
   if err != nil {
       return reconcile.Result{}, err 
   }  

   // Looking for Database CR/CRD 
   // Check the Database Deployments Replicas size
   // If deployment.replicas size != cr.size, then update it
   // Then, go to the beginning of the reconcile
   if err != nil {
       return reconcile.Result{Requeue: true}, nil
   }  
   ...

   // If it is at the end of the loop, then:
   // All was done successfully and the reconcile can stop  
   return reconcile.Result{}, nil

}

```

以下是重新启动协调的可能返回选项:

*   **错误:**

> `return reconcile.Result{}, err`

*   **没有错误:**

> `return reconcile.Result{Requeue: true}, nil`

*   **因此，要停止协调，请使用:**

> `return reconcile.Result{}, nil`

**注:**更多细节，查看`Reconcile`及其`Result` [实现](https://godoc.org/sigs.k8s.io/controller-runtime/pkg/reconcile)。

### 手表()

观察器负责“观察”对象并触发协调。此外，Operator SDK 工具将为每个主要资源(CRD)生成一个监视功能。这里有一个例子:

```
// Watch for changes to primary resource Memcached
err = c.Watch(&source.Kind{Type: &cachev1alpha1.Memcached{}}, &handler.EnqueueRequestForObject{})
if err != nil {
    return err
}

```

通过跟随[入门](https://github.com/operator-framework/getting-started)，还将实现其管理的每个次级对象的一个 watch 函数，如下图。

```
// Watch for changes to secondary resource Pods and requeue the owner Memcached

err = c.Watch(&source.Kind{Type: &appsv1.Deployment{}}, &handler.EnqueueRequestForOwner{
    IsController: true,
    OwnerType:    &cachev1alpha1.Memcached{},
})
if err != nil {
    return err
}

err = c.Watch(&source.Kind{Type: &corev1.Service{}}, &handler.EnqueueRequestForOwner{
    IsController: true,
    OwnerType:    &cachev1alpha1.Memcached{},
})
if err != nil {
    return err
}

```

此外，下面的代码确保了集群上运行的 Memcached 副本的数量。

```
// Ensure the deployment size is the same as the spec
size := memcached.Spec.Size
if *deployment.Spec.Replicas != size {
    deployment.Spec>.Replicas = &size
    err = r.client.Update(context.TODO(), deployment)
    if err != nil {
        reqLogger.Error(err, "Failed to update Deployment.", "Deployment.Namespace", deployment.Namespace, "Deployment.Name", deployment.Name)
         return reconcile.Result{}, err
    }
}

```

之后，您可以通过执行以下步骤来检查上述代码是否正常工作。

1.  放大或缩小 Memcached pod。
2.  检查副本是否会因为上面的代码而恢复到原始大小。

**注意:**以上步骤只有在您能够遵循指南并全部成功完成的情况下才有效。

*Last updated: January 21, 2022*