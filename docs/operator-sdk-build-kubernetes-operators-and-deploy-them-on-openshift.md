# Operator SDK:构建 Kubernetes 操作符并在 OpenShift 上部署它们

> 原文：<https://developers.redhat.com/blog/2020/04/28/operator-sdk-build-kubernetes-operators-and-deploy-them-on-openshift>

Operator SDK 使构建 T2 本地应用变得简单，提供了构建、测试和打包操作符的工具。SDK 还帮助开发人员构建操作符，而不需要了解 Kubernetes API 的复杂性。

在本文中，我们将基于[【Spring Boot】](https://developers.redhat.com/topics/spring-boot/)和 Camel 创建一个示例操作符来部署[一个示例应用程序](https://github.com/shailendra14k/Examples/tree/master/Sample)。这个应用程序是一个简单的骆驼路线，它使用了回流组件。在构建操作符之后，我们将在 OpenShift 集群上部署它。

## 开始

在创建示例操作符之前，我们需要设置示例应用程序(或者，您可以自己创建任何类似的应用程序)。如果你还没有安装[波德曼](https://podman.io/)和[玛文](http://maven.apache.org/)，现在就安装。然后，执行以下操作:

1.  [构建示例应用程序的映像](https://github.com/shailendra14k/Examples/blob/master/Sample/README.md)并将其推送到 Quay repo。
2.  从 Homebrew (macOS)、GitHub 安装`operator-sdk` CLI，或者从 master 编译安装。
3.  为您的操作系统安装正确的 Golang 。
4.  使用`cluster-admin`权限访问 OpenShift 集群。
5.  访问 OpenShift CLI ( `oc`)

通过下面的链接，您可以找到如何完成这些任务的解释。

## 创建示例运算符

现在，要创建我们的示例操作符:

1.  创建`sample-operator`项目:

```
$ mkdir -p $GOPATH/src/github.com/shailendra14k/  // rename to your account.
$ cd $GOPATH/src/github.com/shailendra14k/
$ export GO111MODULE=on
$ operator-sdk new sample-operator
$ cd sample-operator
$ go mod tidy // Install dependencies

```

2.  验证目录结构已创建:

```
├── build
│   ├── bin
│   │   ├── entrypoint
│   │   └── user_setup
│   └── Dockerfile
├── cmd
│   └── manager
│       └── main.go
├── deploy
│   ├── operator.yaml
│   ├── role_binding.yaml
│   ├── role.yaml
│   └── service_account.yaml
├── go.mod
├── go.sum
├── pkg
│   ├── apis
│   │   └── apis.go
│   └── controller
│       └── controller.go
├── tools.go
└── version
    └── version.go

```

**注意:**这里是关于 SDK 项目布局的更多信息。

2.  创建自定义资源定义(CRD):

```
$ operator-sdk add api --api-version=shailendra14k.com/v1alpha1 --kind=Sample

```

3.  打开`pkg/apis/shailendra14k/v1alpha1/sample_types.go`。
4.  更新`Sample`自定义资源规格和`Status`:

```
type SampleSpec struct {
  Size        int32             `json:"size"`
  BodyValue   string            `json:"bodyvalue"`
  Image        string           `json:"image"`

}

// SampleStatus defines the observed state of Sample
type SampleStatus struct {
    Nodes []string `json:"nodes"`
}
```

5.  修改`*_types.go`后，运行以下命令更新生成的代码:

```
$ operator-sdk generate k8s

//Generate the updated CRDs
$ operator-sdk generate crds

```

6.  创建一个新的控制器来监视和协调`Sample`资源:

```
$ operator-sdk add controller --api-version=shailendra14k.com/v1alpha1 --kind=Sample

```

7.  用[`sample_controller.go`](https://github.com/shailendra14k/sample-operator/blob/master/pkg/controller/sample/sample_controller.go)替换默认`pkg/controller/sample/sample_controller.go`。
8.  构建操作员映像并将其推送到注册表中:

```
$ operator-sdk build quay.io/shailendra14k/sample-operator:v0.1 --image-builder podman

//Verify the operator image cretaed locally
$ podman images
REPOSITORY                                                TAG      IMAGE ID       CREATED          SIZE
quay.io/shailendra14k/sample-operator                     v0.1     50fceb91c078   27 seconds ago   150 MB

// Push the image

$ podman push quay.io/shailendra14k/sample-operator:v0.1

```

9.  用正确的图像细节更新默认值`deploy/operator.yaml`:

```
serviceAccountName: sample-operator
containers:
  - name: sample-operator
    # Replace this with the built image name
    image: quay.io/shailendra14k/sample-operator:v0.1
    command:
    - sample-operator
    imagePullPolicy: Always

```

## 在 OpenShift 上部署示例操作符

现在您已经有了一个操作符(`sample-operator`)，通过执行以下操作来部署它:

1.  创建新项目:

```
$ oc new-project sample-operator

```

2.  创建示例 CRD:

```
$ oc create -f deploy/crds/shailendra14k.com_samples_crd.yaml

```

3.  部署操作员并设置基于角色的访问控制(RBAC):

```
$ oc create -f deploy/service_account.yaml
$ oc create -f deploy/role.yaml
$ oc create -f deploy/role_binding.yaml
$ oc create -f deploy/operator.yaml

```

4.  验证部署和操作员日志:

```
$ oc get all
NAME                                   READY     STATUS    RESTARTS   AGE
pod/sample-operator-867cf7c68f-jpjxk   1/1       Running   0          95s

NAME                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/sample-operator-metrics   ClusterIP   172.30.125.171   <none>        8383/TCP,8686/TCP   76s

NAME                              READY     UP-TO-DATE   AVAILABLE   AGE
deployment.apps/sample-operator   1/1       1            1           97s

NAME                                         DESIRED   CURRENT   READY     AGE
replicaset.apps/sample-operator-867cf7c68f   1         1         1         97s

//Operator logs
{"level":"info","ts":1585146465.5662885,"logger":"metrics","msg":"Metrics Service object created","Service.Name":"sample-operator-metrics","Service.Namespace":"sample-operator"}
{"level":"info","ts":1585146468.5416582,"logger":"cmd","msg":"Starting the Cmd."}
{"level":"info","ts":1585146468.5419788,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
{"level":"info","ts":1585146468.5421832,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"sample-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1585146468.6432557,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"sample-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1585146468.743742,"logger":"controller-runtime.controller","msg":"Starting Controller","controller":"sample-controller"}
{"level":"info","ts":1585146468.7437823,"logger":"controller-runtime.controller","msg":"Starting workers","controller":"sample-controller","worker count":1}

```

5.  创建`Sample`自定义资源(CR):

```
$ cat deploy/crds/shailendra14k.com_v1alpha1_sample_cr.yaml 

apiVersion: shailendra14k.com/v1alpha1
kind: Sample
metadata:
  name: example-sample
spec:
  # Add fields here
  size: 2
  bodyvalue: "Response received from POD : {{env:HOSTNAME}}"
  image: "quay.io/shailendra14k/sample:v0.1"

$ oc create -f deploy/crds/shailendra14k.com_v1alpha1_sample_cr.yaml 
sample.shailendra14k.com/example-sample created

```

6.  验证应用程序已部署且 pod 已创建:

```
$ oc get deployment
NAME              READY     UP-TO-DATE   AVAILABLE   AGE
example-sample    2/2       2            2           4m8s
sample-operator   1/1       1            1           10m

$ oc get pods
NAME                               READY     STATUS    RESTARTS   AGE
example-sample-7d856cb9fc-7pbg2    1/1       Running   0          3m29s
example-sample-7d856cb9fc-bjg47    1/1       Running   0          3m29s
sample-operator-867cf7c68f-jpjxk   1/1       Running   0          10m

$ oc get samples
NAME             AGE
example-sample   5m23s

```

7.  创建一个服务和路线来测试 Camel route 应用程序(`Sample` image):

```
$ oc create service clusterip sample --tcp=8080:8080
$ oc expose svc/sample

```

8.  测试应用程序:

```
$ curl http://sample-sampleoperator.apps.lab.com/test
Response received from POD : example-sample-7d856cb9fc-7pbg2

$ curl http://sample-sampleoperator.apps.lab.com/test
Response received from POD : example-sample-7d856cb9fc-bjg47

```

9.  要将应用程序大小从两个更新为一个:

```
apiVersion: shailendra14k.com/v1alpha1
kind: Sample
metadata:
  name: example-sample
spec:
  # Add fields here
  size: 1
  bodyvalue: "Response received from POD : {{env:HOSTNAME}}"
  image: "quay.io/shailendra14k/sample:v0.1"

$ oc apply -f deploy/crds/shailendra14k.com_v1alpha1_sample_cr.yaml

$ oc get deployment
NAME              READY     UP-TO-DATE   AVAILABLE   AGE
example-sample    1/1       1            1           3m1s

```

最后，您可以使用以下命令删除命名空间:

```
$ oc delete project sample-operator

```

## 结论

感谢阅读！我希望这篇文章能够帮助您开始使用 Operator SDK 并在 OpenShift 上部署操作符。

*Last updated: January 19, 2022*