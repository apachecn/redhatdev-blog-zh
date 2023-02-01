# 在 OpenShift 上测试基于内存的水平窗格自动缩放

> 原文：<https://developers.redhat.com/blog/2020/03/19/testing-memory-based-horizontal-pod-autoscaling-on-openshift>

[Red Hat OpenShift](http://developers.redhat.com/openshift/) 主要为 CPU 提供水平 pod 自动缩放(HPA ),但它也可以执行基于内存的 HPA，这对于内存密集型而非 CPU 密集型的应用程序非常有用。在本文中，我演示了如何使用 OpenShift 的基于内存的水平窗格自动缩放功能([技术预览](https://docs.openshift.com/container-platform/3.11/dev_guide/pod_autoscaling.html#pod-autoscaling-memory))在内存需求增加的情况下自动缩放窗格。本文中执行的测试可能不一定反映真实的应用程序。这些测试的目的只是以最简单的方式展示基于内存的 HPA。

我使用一个简单的 PHP 应用程序(`index.php`)，它在每次发出请求时在内存中创建一个大数组。代码如下所示:

```
<?php

$arr = array();
$arr_size = 100000;

for ($i=1;$i<=$arr_size;$i++) {
$arr[] = $i;
}

echo "created an array of $arr_size entries";

?>

```

您可以使用任何语言执行此测试。负载测试通过向应用程序创建多个并行 curl 请求来工作。我选择 PHP 是因为它个人的方便。

## 检查`v2beta1`是否启用

检查您的集群中是否启用了`v2beta1`:

```
# oc get --raw /apis/autoscaling/v2beta1
{"kind":"APIResourceList","apiVersion":"v1","groupVersion":"autoscaling/v2beta1","resources":[{"name":"horizontalpodautoscalers","singularName":"","namespaced":true,"kind":"HorizontalPodAutoscaler","verbs":["create","delete","deletecollection","get","list","patch","update","watch"],"shortNames":["hpa"],"categories":["all"]},{"name":"horizontalpodautoscalers/status","singularName":"","namespaced":true,"kind":"HorizontalPodAutoscaler","verbs":["get","patch","update"]}]}

```

如果没有启用，请按照文档在您各自的 OpenShift 版本上启用它。

## 设置您的测试平台

在测试之前，您需要设置测试环境和测试应用程序。

### 设置您的环境

要设置新环境，请执行以下操作:

1.  创建新项目:

```
# oc new-project memhpa
Now using project "memhpa" on server "https://console.ocp.mylab:8443".

```

2.  创建一个图像流(假设您已经正确设置了身份验证):

```
# oc import-image myphp --insecure --from=registry.redhat.io/openshift3/php-55-rhel7:latest --confirm
imagestream.image.openshift.io/myphp imported

```

3.  在创建第一个应用程序之前，对命名空间应用限制:

```
# cat limits.yml 
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "myphp-resource-limits"
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "2"
        memory: "120Mi"
      min:
        cpu: "200m"
        memory: "6Mi"
    - type: "Container"
      max:
        cpu: "2"
        memory: "120Mi"
      min:
        cpu: "100m"
        memory: "4Mi"
      default:
        cpu: "300m"
        memory: "100Mi"
      defaultRequest:
        cpu: "200m"
        memory: "100Mi"
      maxLimitRequestRatio:
        cpu: "10"

# oc create -f limits.yml
limitrange/myphp-resource-limits created
```

### 设置应用程序

要设置您的应用程序:

1.  创建应用程序:

```
# oc new-app --name app1 myphp:latest~https://gitlab.mylab/myproject/phpapp.git --build-env "GIT_SSL_NO_VERIFY=true"

```

2.  等到您的应用程序构建并运行:

```
# oc get pods
NAME READY STATUS RESTARTS AGE
app1-1-build 0/1 Completed 0 2m
app1-1-plchw 1/1 Running 0 27s

```

3.  创建路线:

```
# oc expose svc app1
route/app1-memhpa.apps.ocp.mylab created

```

## 测试应用程序

现在，为了测试您的应用程序:

1.  测试应用程序:

```
# curl http://app1-memhpa.apps.ocp.mylab/
created an array of 100000

```

如果您的应用程序失败，请尝试减小 PHP 数组的大小。

2.  创建基于内存的 HPA 定义:

```
# cat hpa.yml 
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-metrics-memory
spec:
  scaleTargetRef:
    apiVersion: apps.openshift.io/v1
    kind: DeploymentConfig
    name: app1
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      targetAverageUtilization: 90

# oc create -f hpa.yml
horizontalpodautoscaler.autoscaling/hpa-resource-metrics-memory created

# oc get hpa
NAME                          REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   <unknown>/90%   1         10        0          31m

```

3.  等待，直到上面显示的`<unknown>`值变为整数:

```
# oc get hpa
NAME                          REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   75%/90%   1         10        1          33m

```

4.  创建一个负载。我使用一个简单的脚本来循环执行一个`curl`命令:

```
# cat loadphp.sh
#!/bin/bash

while true; do curl http://app1-memhpa.apps.ocp.mylab/; done

done

```

5.  运行以下命令几次，直到您注意到负载增加:

```
# nohup ./loadphp.sh &
```

## 观察 HPA

您将开始注意到内存利用率的增加和相应的自动伸缩:

```
# oc get hpa
NAME                          REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   75%/90%   1         10        1          33m

# oc get hpa
NAME                          REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   94%/90%   1         10        2          39m

# oc get hpa
NAME                          REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   90%/90%   1         10        2          48m

# oc get hpa
NAME                          REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   85%/90%   1         10        3          52m

```

接下来，停止加载，然后观察 HPA。负载停止几分钟后，自动秤最终会将容器缩小到一个:

```
NAME                          REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-resource-metrics-memory   DeploymentConfig/app1   29%/90%   1         10        1          1h

```

## 结论

在本文中，我的目的是提供最简单的方法来设置和测试基于内存的水平 pod 自动缩放。我用一个 PHP 网页演示了这个过程，这个网页在内存中创建了一个大型数组，这个数组是用一个基本的红帽 S2I PHP 映像构建的，并在一个带有 limits 和 HPA 的名称空间中设置。

设置好环境后，我创建了一个基本的 bash 脚本来给应用程序加载，以便观察内存中不断增加的负载，直到得到多个自动缩放的 pods。停止装载后，几分钟内，自动秤将豆荚减少到一个。

*特别感谢[达蒙·哈切特](https://developers.redhat.com/blog/author/dhatchett/)对本文进行同行评议。*

*Last updated: June 29, 2020*