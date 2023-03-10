# 在 Golang 中创建一个 Kubernetes 操作符来自动管理一个简单的有状态的应用程序

> 原文：<https://developers.redhat.com/blog/2020/12/16/create-a-kubernetes-operator-in-golang-to-automatically-manage-a-simple-stateful-application>

一个 Kubernetes 操作员作为其应用程序的自动化站点可靠性工程师，在软件中编码专家管理员的技能。例如，操作员可以管理一个数据库服务器集群，并配置和管理其应用程序。它还可以安装声明的软件版本和指定数量的成员的数据库集群。在应用程序运行的同时，运营商会继续监控应用程序，并且可以自动备份数据、从故障中恢复，以及随着时间的推移升级应用程序。

集群用户使用 [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) 和其他标准工具与运营商及其应用程序合作，从而扩展 [Kubernetes](/topics/kubernetes/) 服务。运营商使用[定制资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) (CRs)来管理应用及其组件。他们遵循 Kubernetes 原则，特别是通过使用[控制器(控制回路)](https://kubernetes.io/docs/concepts/architecture/controller)。

在本文中，您将学习如何使用 Kubernetes 操作符部署有状态应用程序。在这种情况下，操作员使用 [operator-sdk](https://sdk.operatorframework.io/) 和一个定制资源在 MySQL 上部署一个 WordPress 站点。

**注意**:你对操作符和操作符模式是陌生的吗？[查看 Kubernetes 文档](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)和 [Kubernetes 运营商电子书](/books/kubernetes-operators/)了解更多信息。

## 先决条件示例

要创建 Kubernetes 操作员并使用此演示，首先安装以下软件:

*   [Golang v1.12+](https://golang.org/dl/)
*   [operator-sdk](https://github.com/operator-framework/operator-sdk) 命令行界面(版本 15)
*   一个[迷你库启动](https://kubernetes.io/docs/tasks/tools/install-minikube/)
*   [kubectl 客户端](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## 构建并初始化 Kubernetes 操作符

要构建操作符，从`$GOPATH/src directory`开始。要初始化它，请运行以下命令:

```
operator-sdk new wordpress-operator --type go --repo github.com/<github-user-name>/<github-repo-name>

```

输出表示如下:

```
INFO[0000] Creating new Go operator 'wordpress-operator’.
INFO[0000] Created go.mod
INFO[0000] Created tools.go
INFO[0000] Created cmd/manager/main.go
INFO[0000] Created build/Dockerfile
INFO[0000] Created build/bin/entrypoint
INFO[0000] Created build/bin/user_setup
INFO[0000] Created deploy/service_account.yaml
INFO[0000] Created deploy/role.yaml
INFO[0000] Created deploy/role_binding.yaml
INFO[0000] Created deploy/operator.yaml
INFO[0000] Created pkg/apis/apis.go
INFO[0000] Created pkg/controller/controller.go
INFO[0000] Created version/version.go
INFO[0000] Created .gitignore
INFO[0000] Validating project
```

初始化操作器后，运行`cd wordpress-operator`命令。

## 创建自定义资源定义

[自定义资源定义](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) (CRDs)定义我们与 Kubernetes API 交互的资源。这类似于如何定义现有资源，例如 [pods](https://kubernetes.io/docs/concepts/workloads/pods/) 、[部署](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)、服务、持久卷声明(PVC)等等。在这种情况下，我们指定格式`<group>/<version>`的`api-version`，并创建`kind`自定义资源。要为 WordPress 操作者创建一个 CRD，运行命令:

```
operator-sdk add api --kind Wordpress --api-version example.com/v1
```

现在检查文件:

```
deploy/crds/example.com_v1_wordpress_cr.yaml
```

这是生成类型的 CRD 示例。它预先填充了适当的`api-version`和`kind`以及资源名称。此外，确保 spec 部分使用与我们创建的 CRD 相关的值来完成。检查以下文件:

```
deploy/crds/example.com_wordpresses_crd.yaml
```

该文件是 CRD 清单的开始。SDK 生成许多与资源类型名称相关的字段。在`pkg/apis/example/v1/*_types.go`文件中，寻址两个`struct`对象，称为`spec`对象和`status`对象。检查以下内容:

```
// WordpressSpec defines the desired state of Wordpress  
type WordpressSpec struct s
{                                                                   
    SQLRootPassword string `json:"sqlrootpassword"`// the user will provide the root password through CR
}
```

通过运行`operator-sdk generate crds`和`operator-sdk generate k8s`命令，使用这些更改更新 CRD。这将在`*_types.go`中指定的规格添加到`*crd.yaml`中。

## 设置控制器

在 operator pod 中设置一个控制器来监视自定义资源的变化并做出相应的反应。首先，使用`operator-SDK`生成控制器框架代码，例如:

```
operator-sdk add controller --api-version=example.com/v1 --kind=Wordpress
```

接下来，编辑文件以包含控制器逻辑:

```
pkg/controller/wordpress/wordpress_controller.go
```

### 打造腕表

每次对定制资源进行更改时，都会调用主协调函数。让我们一行一行地检查代码。最初，控制器需要为资源添加观察器，这样 Kubernetes 就可以告诉控制器资源所需的更改。初始观察器是为主要资源`Wordpress`(在我们的例子中)创建的，它由控制器监控。例如:

```
// Watch for changes to primary resource Wordpress
err = c.Watch(&source.Kind{Type: &examplev1.Wordpress{}}, &handler.EnqueueRequestForObject{})
if err != nil {
    return err
}
```

接下来，我们可以为子资源创建后续的观察器，比如 pod、部署、服务、PVC 等等。操作员使用此手表来支持主要资源。通过将`OwnerType`的值指定为主资源，为子资源创建监视。例如:

```
err = c.Watch(&source.Kind{Type: &appsv1.Deployment{}}, &handler.EnqueueRequestForOwner{
      IsController: True,
      OwnerType: &examplev1.Wordpress{},
})
if err != nil {
      return err
}
err = c.Watch(&source.Kind{Type: &corev1.Service{}}, &handler.EnqueueRequestForOwner{
    IsController: true,
    OwnerType: &examplev1.Wordpress{},
})
if err != nil {
   return err
}
err = c.Watch(&source.Kind{Type: &corev1.PersistentVolumeClaim{}}, &handler.EnqueueRequestForOwner{
    IsController: true,
    OwnerType: &examplev1.Wordpress{},
})
if err != nil {
     return err
}
```

### 运行协调循环

现在，运行协调功能，也称为*协调循环*。这是实际逻辑所在的地方。该函数返回`reconcile.Result{}`,它指示协调循环是否需要执行另一遍。基于`reconcile.Result{}`返回值的可能结果是:

| **结果** | **描述** |
| `return reconcile.Result{}, nil` | 协调过程无错误地完成，因此不需要通过协调循环进行另一次迭代。 |
| `return reconcile.Result{}, err` | 由于出现错误，协调失败，Kubernetes 需要将其重新排队并再次运行。 |
| `return reconcile.Result{Requeue: true}, nil` | reconcile 没有遇到错误，但是，Kubernetes 需要将其重新排队并运行另一个迭代。 |

举个例子:

```
return reconcile.Result{RequeueAfter: time.Second*5}, nil
```

将此示例与表的最后一个条目进行比较。在重新运行请求之前，观察器会等待指定的时间，例如 5 秒钟。当我们连续运行多个步骤时，这种方法很有用，尽管它可能需要更长的时间来完成。如果一个后端服务在启动之前需要一个正在运行的数据库，我们可以使用这个例子来重新排队 reconcile 函数，延迟一段时间，以便给数据库启动时间。一旦数据库开始运行，操作员就不会将协调请求重新排队，其余的步骤会继续。我建议查看一下 [Kubernetes API 文档](https://godoc.org/k8s.io/api)，尤其是`core/v1`和`apps/v1`目录，了解更多细节。

### 协调功能

接下来，让我们考虑协调函数的代码。我们再一次一行一行来。最初，协调功能检索主资源。例如:

```
// Fetch the Wordpress instance
wordpress := &examplev1.Wordpress{}
err := r.client.Get(context.TODO(), request.NamespacedName, wordpress) -----1
if err != nil {
      if errors.IsNotFound(err) {
       // Request object not found, could have been deleted after reconcile request.
       // Owned objects are automatically garbage collected. For additional cleanup logic use finalizers.
       // Return and don't requeue
          return reconcile.Result{}, nil
      }
   // Error reading the object - requeue the request.
   return reconcile.Result{}, err
}

// ensure that the child resources are running (example can be seen in below snippet)

//if everything goes fine
return reconcile.Result{}, nil
```

该函数检查`Wordpress`资源是否已经存在。变量`r`是调用协调函数的协调器对象。`client`是 Kubernetes API 的客户。接下来，我们创建一个子资源。与主资源类似，协调器通过调用 Kubernetes `client`的`Get()`来检查子资源是否存在。如果没有，它会在目标名称空间中创建子资源。例如:

```
found := &appsv1.Deployment{}
err := r.client.Get(context.TODO(), types.NamespacedName{
    Name: dep.Name,
    Namespace: instance.Namespace,
}, found)
if err != nil && errors.IsNotFound(err) {

// Create the deployment

        log.Info("Creating a new Deployment", Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
        err = r.client.Create(context.TODO(), dep)   ------------------1

       if err != nil {

        // Deployment failed
         log.Error(err, "Failed to create new Deployment", "Deployment.Namespace", dep.Namespace, "Deployment.Name", dep.Name)
         return &reconcile.Result{}, err
         }
     // Deployment was created successfully

       return nil, nil

}else if err != nil {
    // Error that isn't due to the deployment not existing
     log.Error(err, "Failed to get Deployment")
     return &reconcile.Result{}, err
}
// deployment successful
return nil, nil
```

### MySQL 部署实例

下面是 MySQL 部署实例的代码片段(`dep`)。注意加粗的线条:

```
labels := map[string]string{
"app": cr.Name,
}
matchlabels := map[string]string{
"app": cr.Name,
"tier": "mysql",
}

dep := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
             Name: "wordpress-mysql",
             Namespace: cr.Namespace,
             Labels: labels,
          },

        Spec: appsv1.DeploymentSpec{
            Selector: &metav1.LabelSelector{
                          MatchLabels: matchlabels,
                    },
            Template: corev1.PodTemplateSpec{
                           ObjectMeta: metav1.ObjectMeta{
                           Labels: matchlabels,
                              },
                           Spec: corev1.PodSpec{
                              Containers: []corev1.Container{{
                              Image: "mysql:5.6",
                              Name: "mysql",
                              Env: []corev1.EnvVar{
                                   {
                                     Name: "MYSQL_ROOT_PASSWORD",
                                     Value: cr.Spec.SQLRootPassword,  ------1
                                    },
                               },

                              Ports: []corev1.ContainerPort{{
                                     ContainerPort: 3306,
                                     Name: "mysql",
                                      }},
                              VolumeMounts: []corev1.VolumeMount{
                                            {
                                              Name: "mysql-persistent-storage",
                                              MountPath: "/var/lib/mysql",
                                           }, 
                                       }, 
                                   }, 
                              },

            Volumes: []corev1.Volume{
                          {
                              Name: "mysql-persistent-storage",
                              VolumeSource: corev1.VolumeSource{
                                            PersistentVolumeClaim: &corev1.PersistentVolumeClaimVolumeSource{
                                                      ClaimName: "mysql-pv-claim",
                                                 },
                                             },
                                      },
                              },
                       },
                },
         },
}

controllerutil.SetControllerReference(cr, dep, r.scheme) -------2
```

注意加粗的线条:

1.  重要的是要注意从`cr.Spec`中获取的`MYSQL_ROOT_PASSWORD`的值。
2.  这是定义中最关键的一行，因为它建立了主资源`Wordpress`和子资源 deployment 之间的父子关系。我们还可以为 pod、deployment、service、PVC 等子资源编写类似的代码。我建议你也查看一下 [WordPress-Operator](https://github.com/priyanka19-98/Wordpress-Operator) 以获得更多细节。

## 运行 Kubernetes WordPress 操作器

现在运行 WordPress 操作器。首先，使用以下命令确保 minikube 集群正在运行:

```
kubectl create-f ./deploy/crds/example.com_wordpresses_crd.yaml
```

```
operator-sdk run --local
```

在下一次终端运行中，使用以下命令:

```
kubectl apply -f ./deploy/crds/example.com_v1_wordpress_cr.yaml
```

完成后，将显示以下日志:

```
INFO[0000] Running the operator locally in namespace default. 
{"level":"info","ts":1598973876.2819793,"logger":"cmd","msg":"Operator Version: 0.0.1"}
{"level":"info","ts":1598973876.2820053,"logger":"cmd","msg":"Go Version: go1.13.10"}
{"level":"info","ts":1598973876.282011,"logger":"cmd","msg":"Go OS/Arch: linux/amd64"}
{"level":"info","ts":1598973876.2820172,"logger":"cmd","msg":"Version of operator-sdk: v0.15.2"}
{"level":"info","ts":1598973876.285575,"logger":"leader","msg":"Trying to become the leader."}
{"level":"info","ts":1598973876.285611,"logger":"leader","msg":"Skipping leader election; not running in a cluster."}
{"level":"info","ts":1598973876.5921307,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"0.0.0.0:8383"}
{"level":"info","ts":1598973876.596543,"logger":"cmd","msg":"Registering Components."}
{"level":"info","ts":1598973876.5967476,"logger":"cmd","msg":"Skipping CR metrics server creation; not running in a cluster."}
{"level":"info","ts":1598973876.5967603,"logger":"cmd","msg":"Starting the Cmd."}
{"level":"info","ts":1598973876.5973437,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"wordpress-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1598973876.5975914,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"wordpress-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1598973876.5977812,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"wordpress-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1598973876.5979419,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"wordpress-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1598973876.5980544,"logger":"controller-runtime.controller","msg":"Starting Controller","controller":"wordpress-controller"}
{"level":"info","ts":1598973876.598183,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
{"level":"info","ts":1598973876.6982796,"logger":"controller-runtime.controller","msg":"Starting workers","controller":"wordpress-controller","worker count":1}
{"level":"info","ts":1598973876.6983802,"logger":"controller_wordpress","msg":"Reconciling Wordpress","Request.Namespace":"default","Request.Name":"example-wordpress"}
{"level":"info","ts":1598973876.6984997,"logger":"controller_wordpress","msg":"Creating a new PVC","PVC.Namespace":"default","PVC.Name":"wp-pv-claim"}
{"level":"info","ts":1598973876.7138047,"logger":"controller_wordpress","msg":"Creating a new Deployment","Deployment.Namespace":"default","Deployment.Name":"wordpress"}
{"level":"info","ts":1598973876.736821,"logger":"controller_wordpress","msg":"Creating a new Service","Service.Namespace":"default","Service.Name":"wordpress"}
{"level":"info","ts":1598973876.8298655,"logger":"controller_wordpress","msg":"Reconciling Wordpress","Request.Namespace":"default","Request.Name":"example-wordpress"}
{"level":"info","ts":1598973876.8301716,"logger":"controller_wordpress","msg":"Creating a new Service","Service.Namespace":"default","Service.Name":"wordpress"}
```

此示例还显示了 pod、部署、服务、PVC 等，如下所示:

```
[pjiandan@pjiandan crds]$ kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
wordpress-6d5b4988ff-dcxfj         1/1     Running   0          16h
wordpress-mysql-59d5d89ff8-qj92r   1/1     Running   0          17h
[pjiandan@pjiandan crds]$ kubectl get svc
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1       <none>        443/TCP        19h
wordpress         NodePort    10.100.123.86   <none>        80:31881/TCP   16h
wordpress-mysql   ClusterIP   None            <none>        3306/TCP       17h
[pjiandan@pjiandan crds]$ kubectl get deploy
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
wordpress         1/1     1            1           16h
wordpress-mysql   1/1     1            1           17h
[pjiandan@pjiandan crds]$ kubectl get pvc
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    pvc-9ee52dce-b7b7-433d-8596-22392033e55e   10Gi       RWO            standard       17h
wp-pv-claim      Bound    pvc-8674f3fa-acb3-4cd7-9283-5ecec8305945   10Gi       RWO            standard       16h

```

接下来，运行下面的命令来返回 WordPress 服务的 IP 地址:

```
minikube service wordpress --url
```

此 IP 地址响应的示例如下:

```
http://192.168.99.101:31881
```

## 验证站点正在运行

最后，复制 IP 地址并在浏览器中加载页面以查看站点，如图 1 所示。

[![Initial WordPress page to load to browser .](img/4cd11d774a1fd2792b6885c78aa0c5c7.png)](/sites/default/files/WordPress.png)Figure 1: Copy the IP address and load the WordPress page to the browser.

## 结论

这就完成了我的演示和介绍！在本文中，我演示了如何使用 Kubernetes 操作符来部署有状态应用程序。这个操作者使用定制资源使用`operator-sdk`项目在 MySQL 上部署 WordPress。如果你需要部署一个没有操作符的有状态应用，请看:[例子:用持久卷部署 WordPress 和 MySQL】。](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)

*Last updated: August 3, 2021*