# Kubernetes 配置模式，第 2 部分:Kubernetes 控制器的模式

> 原文：<https://developers.redhat.com/blog/2021/05/05/kubernetes-configuration-patterns-part-2-patterns-for-kubernetes-controllers>

本文是关于 Kubernetes 配置模式的两部分文章系列的第二部分，您可以用它来配置您的 Kubernetes 应用程序和控制器。第一篇文章[介绍了只使用 Kubernetes 原语的模式和反模式](/blog/2021/04/28/kubernetes-configuration-patterns-part-1-patterns-for-kubernetes-primitives/)。这些简单的模式适用于任何应用程序。第二篇文章描述了需要针对 Kubernetes API 进行编码的更高级的模式，这是 Kubernetes 控制器应该使用的。

您将在本文中学习的模式适用于基本 Kubernetes 特性不够的场景。当您无法将另一个名称空间中的`ConfigMap`挂载到`Pod`中，无法在不杀死`Pod`的情况下重新加载配置等等时，这些模式将会对您有所帮助。

和第一篇文章一样，为了简单起见，我在示例 YAML 文件中只使用了`Deployment` s。然而，这些示例应该与其他 *PodSpecables* (描述`PodSpec`的任何东西)一起工作，例如`DaemonSet` s 和`ReplicaSet` s。我还省略了像`image`、`imagePullPolicy`这样的字段，以及示例`Deployment` YAML 中的其他字段。

## 使用集中配置图进行配置

当`ConfigMap`和`Deployment`、`Pod`、`ReplicaSet`或其他组件位于不同的名称空间时，不可能将它们挂载到这些组件上。但是在某些情况下，对于运行在不同名称空间中的组件，您需要一个单一的中央配置存储。下面的`Deployment`说明了这种情况:

```
apiVersion: v1
kind: ConfigMap
metadata:
    name: logcollector-config
    namespace: logcollector
data:
    buffer: "2048"
    target: "file"
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: logcollector-agent
    namespace: user-ns-1
spec:
    ...
    template:
        spec:
            containers:
            - name: server
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: logcollector-agent
    namespace: user-ns-2
spec:
    ...
    template:
        spec:
            containers:
            - name: server

```

在这个例子中，假设有一个中央日志收集器系统，如图 1 所示。Kubernetes 控制器(示例中未显示)为每个名称空间创建一个单独的日志收集器代理`Deployment`。这就是为什么`ConfigMap`和`Deployment`名称空间是不同的。

[![In this Kubernetes controller pattern, different containers in different namespaces retrieve their configuration by reading from a centralized ConfigMap.](img/008d737ae15b763e0a8c7ac5a97d6b4f.png "advanced_1")](/sites/default/files/blog/2020/11/advanced_1.png)Figure 1:

Figure 1: Different containers read from a centralized ConfigMap.

在每个名称空间内的日志收集器代理中，代理应用程序读取中央`ConfigMap`进行配置。请注意`ConfigMap`没有安装到容器上。应用程序需要使用 Kubernetes API 来读取`ConfigMap`，如下伪代码所示:

```
package main

import ...

func main(){
  if _, err := kubeclient.Get(ctx).CoreV1().ConfigMaps("logcollector").Get("logcollector-config", metav1.GetOptions{}); err == nil {
    watchConfigmap("logcollector" ,"logcollector-config", func(configMap *v1.ConfigMap) {
      updateConfig(configMap)
    })
  } else if apierrors.IsNotFound(err) {
      log.Fatal("Central ConfigMap 'logcollector-config' in namespace 'logcollector' does not exist")
  } else {
      log.Fatal("Error reading central ConfigMap 'logcollector-config' in namespace 'logcollector'")
  }
}

```

这段伪代码获取`ConfigMap`并开始观察它。如果[配置图](#advanced_1)有任何变化，该功能将更新配置。没有必要推出新的`Deployment`。

由于`ConfigMap`在另一个名称空间中，容器需要额外的权限来获取、读取和查看它。系统管理员应该处理容器的基于角色的访问控制(RBAC)设置。为了简洁起见，这里没有显示这些设置。

## 具有中央和命名空间配置图的配置

带有中央配置映射模式的[配置允许您使用中央配置。在许多情况下，您还需要覆盖每个名称空间的部分配置。](#advanced_1)

除了中心配置之外，这个模式还使用了一个*命名空间配置*。代码中的命名空间`ConfigMap`需要一个单独的表，如图 2 所示。

[![Alt](img/9fbbb21bff517d738aeba9a89a795476.png "advanced_2")](/sites/default/files/blog/2020/11/advanced_2.png)Figure 2:

Figure 2: Each container takes its config from a centralized configuration and a local ConfigMap.

下面是设置名称空间和配置的`Deployment`:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: logcollector-config
  namespace: logcollector
data:
  buffer: "2048"
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: logcollector-config
  namespace: user-ns-1
data:
  buffer: "1024"
  target: "file"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: logcollector
  name: logcollector-deployment
  namespace: user-ns-1
spec:
  ...
  template:
    spec:
      containers:
      - name: server
        env:
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace

```

为了简洁起见，名称空间`user-ns-2`中的`Deployment`没有显示在前面的 YAML 中。它与名称空间`user-ns-1`中的`Deployment`完全相同。

现在有两个`ConfigMap`。一个是在被认为是中央配置的`logcollector`名称空间中。另一个在与日志收集器实例相同的名称空间中，位于`user-ns-1`名称空间中。检索配置的代码是:

```
package main

import ...

func main(){
  if namespace := os.Getenv("NAMESPACE"); ns == "" {
    panic("Unable to determine application namespace")
  }

  var central *v1.ConfigMap
  var local *v1.ConfigMap

  if _, err := kubeclient.Get(ctx).CoreV1().ConfigMaps("logcollector").Get("logcollector-config", metav1.GetOptions{}); err == nil {
    watchConfigmap("logcollector" ,"logcollector-config", func(configMap *v1.ConfigMap) {
      central = configMap
      config := mergeConfig(central, local)
      updateConfig(config)
    })
  } else if apierrors.IsNotFound(err) {
    log.Fatal("Central ConfigMap 'logcollector-config' in namespace 'logcollector' does not exist")
  } else {
    log.Fatal("Error reading central ConfigMap 'logcollector-config' in namespace 'logcollector'")
  }

  if _, err := kubeclient.Get(ctx).CoreV1().ConfigMaps(namespace).Get("logcollector-config", metav1.GetOptions{}); err == nil {
    watchConfigmap(namespace ,"logcollector-config", func(configMap *v1.ConfigMap) {
      central = configMap
      config := mergeConfig(central, local)
      updateConfig(config)
    })
  } else if apierrors.IsNotFound(err) {
    log.Infof("Local ConfigMap 'logcollector-config' in namespace '%s' does not exist", namespace)
  } else {
    log.Fatalf("Error reading local ConfigMap 'logcollector-config' in namespace '%s'", namespace)
  }
}

func mergeConfig(central, local *v1.ConfigMap) (...){
  // merge 2 configmaps here and return the result
}

```

Kubernetes 控制器应用程序将首先检查中央配置，然后检查命名空间配置。命名空间`ConfigMap`中的配置具有优先权。

## 关于模式的注释

没有任何本地`ConfigMap`是可以的。与缺少中央`ConfigMap`不同，如果本地`ConfigMap`不存在，程序不会停止执行。

应用程序需要知道它正在其中运行的容器名称空间，因为本地`ConfigMap`不是使用`ConfigMap`挂载的，并且应用程序需要为本地`ConfigMap`发出一个 GET。因此，应用程序需要告诉 Kubernetes API 它正在为`ConfigMap`寻找的名称空间。

也可以简单地将命名空间`ConfigMap`安装到日志收集器容器中。但是在这种情况下，您将失去无需重新启动就能重新加载配置的能力。

## 配置定制资源

您可以使用定制资源来扩展 Kubernetes API。自定义资源是一个非常强大的概念，但是太复杂了，无法在这里深入解释。下面是一个示例自定义资源:

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: eventconsumers.example.com
spec:
  group: example.com
    versions:
    - name: v1
...
  scope: Namespaced
  names:
    plural: eventconsumers
    singular: eventconsumer
    kind: EventConsumer
...

```

在应用这个定制资源定义之后，您可以在 Kubernetes 上创建一个`EventConsumer`资源:

```
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-1
  namespace: user-ns-1
spec:
  source: source1.foobar.com:443
---
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-2
  namespace: user-ns-2
spec:
  source: source2.foobar.com:443

```

定制资源可以用作配置，由 Kubernetes 控制器读取，如图 3 和图 4 所示。图 3 显示控制器读取定制资源。

[![alt](img/a1fc907ddf8889bb5478c7b8d12b0ce6.png "advanced_3_a")](/sites/default/files/blog/2020/11/advanced_3_a.png)Figure 3:

Figure 3: The controller reads each custom resource.

图 4 显示了在控制器读取和处理定制资源之后，它为每个定制资源创建一个事件消费者应用程序实例。

[![alt](img/ea7890e6a969ea0981b12922b1ce63b0.png "advanced_3_b")](/sites/default/files/blog/2020/11/advanced_3_b.png)Figure 4:

Figure 4: The controller creates event consumer application instances for each custom resource.

在这种情况下，`EventConsumer`定制资源为事件消费者 Kubernetes 控制器创建的事件消费应用程序提供配置。

注意定制资源定义中的`scope: Namespaced`键和值。这告诉 Kubernetes，为这个定义创建的定制资源将存在于一个名称空间中。

为配置中的字段提供强大的 API 是一个很好的特性。有了自定义资源定义中定义的规则，您就可以毫无困难地使用 Kubernetes 验证。

## 使用自定义资源进行配置的缺点

自定义资源不如`ConfigMap`灵活。可以在`ConfigMap`中添加任何形状的数据。还有，不需要在 Kubernetes 注册一个`ConfigMap`；你只是创造了它。您必须通过创建一个`CustomResourceDefinition`来注册定制资源。您在自定义资源中添加的信息应该与在`CustomResourceDefinition`中定义的形状相匹配。如前所述，强大的 API 通常被认为是最佳实践。尽管`ConfigMap` s 更加灵活，但尽可能使用定义明确的形状进行配置是一个好主意。

此外，并不是所有的事情都可以事先明确说明。想象一个可以连接多个数据库的程序。数据库客户机将具有不同的配置，它们的形状也将不同。不可能简单地在定制资源中定义字段，将它们作为一个整体传递给数据库客户机。创建具有超大形状定义的自定义资源并不是一个好主意，因为它可以位于多个形状中。

此外，必须小心维护自定义资源定义，如果不发布自定义资源的新版本，就不能简单地删除字段或以不兼容的方式更新字段。添加新字段没有问题。

## 使用自定义资源的配置，回退到配置图

这种模式是带有定制资源的[配置](#advanced_3)模式和带有中央和命名空间配置映射模式的[配置的混合。](#advanced_2)

在这种模式中，不能在每个场景中使用的配置选项被放在自定义资源中。不同定制资源共有的选项被放在`ConfigMap`中，这样它们就可以被共享。这种模式如图 5 和图 6 所示。图 5 显示控制器正在观察和读取用户命名空间`ConfigMap`和定制资源。然而，它首先依赖于定制资源，对于定制资源中缺少的值，它返回到`ConfigMap`。

[![alt](img/a32bcdf5ae3638642eee65a79712ef07.png "advanced_4_a")](/sites/default/files/blog/2020/11/advanced_4_a.png)Figure 5:

Figure 5: The controller creates the application landscape based on custom resources and ConfigMaps.

图 6 显示了控制器为图 5 中给出的配置创建的应用程序场景。

[![alt](img/5762ca6c3cea8f2db47bf916806a7541.png "advanced_4_b")](/sites/default/files/blog/2020/11/advanced_4_b.png)Figure 6:

Figure 6: The configuration relies first on custom resources, falling back to ConfigMaps.

下面是设置配置的`Deployment`，从定制资源定义开始:

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: eventconsumers.example.com
spec:
  group: example.com
  versions:
  - name: v1
    ...
  scope: Namespaced
  names:
    plural: eventconsumers
    singular: eventconsumer
    kind: EventConsumer
    ...
---
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-1
  namespace: user-ns-1
spec:
  source: source1.foobar.com:443
---
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-2
  namespace: user-ns-2
spec:
  source: source2.foobar.com:443
  privateKey: |
    -----BEGIN PRIVATE KEY-----
    foobarbazfoobar
    barfoobarbazfoo
    bazfoobarfoobar
    -----END PRIVATE KEY-----

```

这里是`ConfigMap`:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: eventconsumer-config
  namespace: eventconsumer
data:
  buffer: "2048"
  privateKey: |
    -----BEGIN PRIVATE KEY-----
    MIICdQIBADANBgk
    nC45zqIvd1QXloq
    bokumO0HhjqI12a
    -----END PRIVATE KEY-----
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: eventconsumer-config
  namespace: user-ns-1
data:
  buffer: "4096"

```

在这个例子中，`EventConsumer` `consumer-1`连接到`source1.foobar.com:8443` URL 来消费事件。不过，请注意，`consumer-2`正在连接到另一个 URL。

`eventconsumer-config`中的中央`ConfigMap`包含缓冲区大小和私钥。这些将由事件消费者控制器使用，除非在本地`ConfigMap`或定制资源中有覆盖。

在`user-ns-1`名称空间中，局部`ConfigMap`覆盖该名称空间中`EventConsumer`的缓冲区大小。对于`consumer-2`不存在本地`ConfigMap`，并且在定制资源中不存在缓冲区大小覆盖，因此中央`ConfigMap`中的默认缓冲区大小将用于该消费者。

在`consumer-2`自定义资源中，定义了一个`privateKey`，所以不使用中央`ConfigMap`中的那个。

## 关于模式的注释

这种模式有助于共享定制资源的公共信息。缓冲区大小或私钥可能已经被硬编码在应用程序中。相反，在这种情况下，甚至默认值都是可配置的。

正如在具有中央和命名空间 ConfigMaps 模式的[配置中所解释的，更好的方法可能是创建一个单独的中央配置定制资源，该资源是集群范围的并且包含共享配置。这可能会额外带来命名空间配置定制资源，等等。如果有许多选项，并且构建和验证起来很复杂，那么使用单独的定制资源可能比使用集中配置定制资源更好，因为它可以利用 Kubernetes 中的 OpenAPI 验证。](#advanced_2)

## 自定义资源中对配置映射的引用

前面的模式[使用定制资源进行配置，退回到 ConfigMaps](#advanced_4) ，有助于共享多个定制资源的配置。但是它不允许您为同一名称空间中的自定义资源指定不同的共享配置。

我们可以用这种模式来解决这个问题，这种模式在`ConfigMap`中保留了长而复杂的配置，如图 7 和图 8 所示。将配置存储在自定义资源规范中是不可取的。

[![alt](img/393e7c75b9c9a4658c0e61901a75f39f.png "advanced_5_a")](/sites/default/files/blog/2020/11/advanced_5_a.png)Figure 7:

Figure 7: Custom resources refer to a ConfigMap for part of their configuration.

图 8 显示了控制器为上图中给出的配置创建的应用程序场景。

[![alt](img/b8b841b553e12cd9bcdfaacf3ce23bbc.png "advanced_5_b")](/sites/default/files/blog/2020/11/advanced_5_b.png)Figure 8:

Figure 8: The controller uses custom resources referring to a ConfigMap to create the application landscape.

下面是设置配置的`Deployment`，从定制资源定义开始:

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: eventconsumers.example.com
spec:
  group: example.com
  versions:
  - name: v1
    ...
  scope: Namespaced
    names:
    plural: eventconsumers
    singular: eventconsumer
    kind: EventConsumer
    ...
---
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-1-a
spec:
  source: source1a.foobar.com:443
  connectionConfig:
  ref: throttled
---
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-1-b
spec:
  source: source1b.foobar.com:443
  connectionConfig:
  ref: throttled
---
apiVersion: example.com/v1
kind: EventConsumer
metadata:
  name: consumer-2
spec:
  source: source2.foobar.com:443
  connectionConfig:
  ref: unlimited

```

这里是`ConfigMap`:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: throttled
data:
  lotsOfConfig: here
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: unlimited
data:
  otherTypeOfConfig: here

```

在这个模式中，定制资源包含对`ConfigMap`的引用。在同一个名称空间中，可以在多个自定义资源中引用`ConfigMap`。也有可能在相同名称空间的定制资源中引用不同的`ConfigMap`。

此外，`ConfigMap`中的配置甚至可以在不同类型的自定义资源中引用。例如，TLS 设置可以保存在一个`ConfigMap`中，并在 Kafka 客户机组件定制资源和 CouchDB 客户机组件定制资源中引用。

## 结论

第一篇文章中的模式和这篇文章完成了这个关于 Kubernetes 配置模式的系列文章。我个人在我参与的项目中看到了更多用于 Kubernetes 控制器的配置模式，有时甚至非常奇怪。这篇文章中精选的模式是我发现的最好的。它们足够灵活，可以让您在它们的基础上解决配置问题。

*Last updated: October 14, 2022*