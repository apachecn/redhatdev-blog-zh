# 如何在 Operator SDK 项目中使用第三方 API

> 原文：<https://developers.redhat.com/blog/2020/02/04/how-to-use-third-party-apis-in-operator-sdk-projects>

[Operator Framework](https://coreos.com/blog/introducing-operator-framework) 是一个用于管理 Kubernetes-native 应用程序的开源工具包。该框架及其特性提供了开发简化复杂性的工具的能力，例如在 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 和 [Red Hat OpenShift](https://developers.redhat.com/openshift/) 上安装、配置、管理和打包应用程序。在本文中，我们展示了如何在 [Operator-SDK](https://github.com/operator-framework/operator-sdk) 项目中使用第三方 API。

在使用 Operator-SDK 构建的项目中，默认情况下只添加 Kubernetes API 模式。但是，您可能需要创建、读取、更新或删除来自另一个 API 的资源——甚至是您自己通过其他 Operator 项目创建的资源。

让我们来看看一个示例场景:如何从 OpenShift API 为 Operator-SDK 项目创建一个 [Route](https://docs.openshift.com/container-platform/4.2/networking/routes/route-configuration.html) 资源。

## 步骤 1:获取 API 模块

在您的项目目录中，通过命令行运行以下命令来获取 OpenShift API 模块:

```
$ go get -u github.com/openshift/api
```

## 步骤 2:使用发现 API 查看新 API 是否存在

此时最好的方法是确保资源在集群中可用，因为我们使用的是第三方 API。您可以通过阅读 [*了解如何检查资源是否可用。为什么不将一个操作者的逻辑与一个特定的 Kubernetes 平台结合起来呢？*](https://developers.redhat.com/blog/2020/01/22/why-not-couple-an-operators-logic-to-a-specific-kubernetes-platform/)

**注意:**如果你正在构建的东西必须运行在特定的 Kubernetes 平台上(例如 OpenShift ),要注意用户可能仍然会尝试使用其他 Kubernetes 平台供应商的项目。例如，他们可能会尝试用 Minikube 检查您的运营商。在这种情况下，如果您没有充分实现项目，您的项目可能会失败，因为 OpenShift APIs 将不存在。最佳实践建议您创建两种情况下都支持的操作员项目。在这个例子中，我们可以使用 [v1。如果集群中有路由](https://docs.okd.io/latest/rest_api/apis-route.openshift.io/v1.Route.html)，或者我们可以创建一个[入口](https://kubernetes.io/docs/concepts/services-networking/ingress)。

## 步骤 3:向方案注册 API

在`main.go`文件中，在行`Setup all Controllers`之前添加模式:

```
    ...
    // Adding the routev1
	if err := routev1.AddToScheme(mgr.GetScheme()); err != nil {
		log.Error(err, "")
		os.Exit(1)
	}

    // Setup all Controllers
	if err := controller.AddToManager(mgr); err != nil {
		log.Error(err, "")
		os.Exit(1)
	}
    ...

```

请注意，您还需要导入模块:

```
import (
	...
	routev1 "github.com/openshift/api/route/v1"
	...
)
```

## 步骤 4:在控制器中使用 API

现在，我们可以在`controller.go`文件的`Reconcile`函数中创建[路线](https://docs.openshift.com/container-platform/4.3/networking/routes/route-configuration.html)资源。继续我们的例子:

```
        ...
        route := &routev1.Route{}
        err = r.client.Get(context.TODO(), types.NamespacedName{Name: memcached.Name, Namespace: memcached.Namespace}, route)
        if err != nil && errors.IsNotFound(err) {
	    // Define a new Rooute object
	    route = r.routeForMemcached(memcached)
	    reqLogger.Info("Creating a new Route.", "Route.Namespace", route.Namespace, "Route.Name", route.Name)
            err = r.client.Create(context.TODO(), route)
            if err != nil {
		    reqLogger.Error(err, "Failed to create new Route.", "Route.Namespace", route.Namespace, "Route.Name", route.Name)
		    return reconcile.Result{}, err			
            }
	} else if err != nil {
	    reqLogger.Error(err, "Failed to get Route.")
	    return reconcile.Result{}, err
	}
        ...
```

创建路线本身:

```
// routeForMemcached returns the route resource
func (r *ReconcileMemcached) routeForMemcached(m *cachev1alpha1.Memcached) *routev1.Route {

	ls := labelsForMemcached(m.Name)
	route := &routev1.Route{
		ObjectMeta: metav1.ObjectMeta{
			Name:      m.Name,
			Namespace: m.Namespace,
			Labels:    ls,
		},
		Spec: routev1.RouteSpec{
			To: routev1.RouteTargetReference{
				Kind: "Service",
				Name: m.Name,
			},
			Port: &routev1.RoutePort{
				TargetPort: intstr.FromString(m.Name),
			},
			TLS: &routev1.TLSConfig{
				Termination: routev1.TLSTerminationEdge,
			},
		},
	}

	// Set MobileSecurityService mss as the owner and controller
	controllerutil.SetControllerReference(m, route, r.scheme)
	return route
}
```

此外，如果此资源发生任何更改，也可以重新触发协调:

```
	err = c.Watch(&source.Kind{Type: &routev1.Route{}}, &handler.EnqueueRequestForOwner{
		IsController: true,
		OwnerType:    &cachev1alpha1.Memcached{},
	})
	if err != nil {
		return err
	}

```

## 步骤 5:使用 API 实现单元测试

您需要开发测试来检查您的实现，以便将第三方模式添加到由[操作符-SDK](https://github.com/operator-framework/operator-sdk) 测试框架提供的[假客户端](https://godoc.org/sigs.k8s.io/controller-runtime/pkg/client/fake)所使用的管理器中。请参见以下示例:

```
func buildReconcileWithFakeClient(objs []runtime.Object, t *testing.T) *ReconcileMemcached {
	s := scheme.Scheme

	// Add route Openshift scheme
	if err := routev1.AddToScheme(s); err != nil {
		t.Fatalf("Unable to add route scheme: (%v)", err)
	}

	s.AddKnownTypes(&v1alpha1.Memcached{}, &v1alpha1.AppService{})

	// create a fake client to mock API calls with the mock objects
	cl := fake.NewFakeClient(objs...)

	// create a ReconcileMemcached object with the scheme and fake client
	return &ReconcileMemcached{client: cl, scheme: s}
}
```

使用上述函数执行测试，如下所示:

```
func TestReconcileMemcached(t *testing.T) {
	type fields struct {
		scheme *runtime.Scheme
	}
	type args struct {
		instance *1alpha1.Memcached
		kind     string
	}
	tests := []struct {
		name      string
		fields    fields
		args      args
		want      reconcile.Result
		wantErr   bool
		wantPanic bool
	}{
		// TODO: Tests
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {

			objs := []runtime.Object{tt.args.instance}
			r := buildReconcileWithFakeClient(objs, t)

            if (err != nil) != tt.wantErr {
				t.Errorf("TestReconcileMemcached error = %v, wantErr %v", err, tt.wantErr)
				return
			}
		})
	}
}
```

现在，您知道了如何在您的操作符项目中使用第三方 API。不仅如此，您还对代码的重用能力有很好的了解。

我要感谢@Joe Lanford，他也为本文提供了反馈和意见。

*Last updated: June 29, 2020*