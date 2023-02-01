# 使用 ConfigMap 在 Kubernetes 上配置 Spring Boot

> 原文：<https://developers.redhat.com/blog/2017/10/03/configuring-spring-boot-kubernetes-configmap>

[ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 是 Spring Boot [外部化](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)配置的 Kubernetes 对应物。 [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 是一个简单的键/值存储，可以将简单的值存储到文件中。在这篇文章“用 ConfigMap 在 Kubernetes 上配置 Spring Boot”中，我们将看到如何使用 [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 来具体化应用程序配置。

在 kubernetes 上配置 spring boot 应用程序的方法之一是使用 [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 。 [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 是一种将特定于应用程序的构件从容器映像中分离出来的方法，从而支持更好的可移植性和外部化。

这篇博文的来源可以在我的 [github repo](http://bit.ly/spring-boot-configmaps-demo) 中找到。在这篇博文中，我们将构建一个简单的 GreeterApplication，它公开了一个 REST API 来问候用户。GreeterApplication 将使用[配置映射](https://developers.redhat.com/search?t=ConfigMap)来具体化应用程序属性。

## 设置

您可能需要访问 Kubernetes 集群才能使用这个应用程序。启动和运行本地 Kubernetes 集群的最简单方法是使用 [minikube。](https://github.com/kubernetes/minikube)博客的其余部分假设你已经启动并运行了 [minikube](https://github.com/kubernetes/minikube) 。

有两种方法可以使用[配置图](https://developers.redhat.com/search?t=ConfigMap)，

1.  [作为环境变量的配置图](#cm-as-env-var)
2.  [将配置图安装为文件](#cm-as-files)

### 作为环境变量的配置映射

假设您已经克隆了我的 [github repo](http://bit.ly/spring-boot-configmaps-demo) ，让我们在本文中将源代码的克隆位置称为$ **PROJECT_HOME** 。

您会注意到 com . red hat . developers . GREETER controller 有一个代码来查找环境变量 **GREETER_PREFIX。**

```
package com.redhat.developers;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class GreeterController {

    @Value("${greeter.message}")
    private String greeterMessageFormat; 

    @GetMapping("/greet/{user}")
    public String greet(@PathVariable("user") String user) {
        String prefix = System.getenv().getOrDefault("GREETING_PREFIX", "Hi");
        log.info("Prefix :{} and User:{}", prefix, user);
        if (prefix == null) {
            prefix = "Hello!";
        }

        return String.format(greeterMessageFormat, prefix, user);
    }
}
```

按照惯例，Spring Boot 应用程序(而不是任何 Java 应用程序)通过系统属性传递这些类型的值。现在，让我们看看如何在 Kubernetes 部署中做到这一点。

*   让我们创建一个 Kubernetes[config maps](https://developers.redhat.com/search?t=ConfigMap)来保存名为 *greeter.prefix* ， 的属性，该属性将通过名为**GREETER _ PREFIX**的环境变量注入到 Kubernetes 部署中。

#### 创建配置图

```
kubectl create configmap spring-boot-configmaps-demo --from-literal=greeter.prefix="Hello"
```

*   您可以使用命令`kubectl get configmap spring-boot-configmaps-demo-oyaml`查看配置图的内容

#### 创建片段部署

一旦我们创建了 Kubernetes [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) ，我们就需要将 **GREETER_PREFIX** 作为环境变量注入到 Kubernetes 部署中。下面的代码片段展示了如何在 kubernetes deployment.yaml 中定义环境变量。

```
spec:
  template:
    spec:
      containers:
        - env:
          - name: GREETING_PREFIX
            valueFrom:
             configMapKeyRef:
                name: spring-boot-configmaps-demo
                key: greeter.prefix
```

*   上面的代码片段定义了一个名为 **GREETING_PREFIX** 的环境变量，它的值将从 config map*spring-boot-config maps-demo*key*greeter . PREFIX .*中设置

**注**:

由于应用程序被配置为使用 [fabric8-maven-plugin](https://maven.fabric8.io) ，我们可以在“$PROJECT_HOME/src/main/fabric8”中创建 Kubernetes 部署和服务片段。 [fabric8-maven-plugin](https://maven.fabric8.io) 负责通过在部署期间合并来自‘$ PROJECT _ HOME/src/main/fabric 8’的片段内容来构建完整的 Kubernetes 清单。

#### 部署应用程序

要部署应用程序，请从＄PROJECT _ HOME`./mvnw clean fabric8:deploy`中执行以下命令。

#### 访问应用程序

应用程序的状态可以用命令检查，`kubectl get pods -w`一旦应用程序被部署，让我们做一个简单的卷曲像`curl $(minikube service spring-boot-configmaps-demo --url)/greet/jerry; echo "";`命令将返回一个消息**你好杰里！欢迎在 Kubernetes 上配置 Spring Boot！**返回消息有一个名为“Hello”的前缀，这是我们通过环境变量 **GREETING_PREFIX** 用 ConfigMap 属性“greeter.prefix”的值注入的。

### 将[配置图](https://kubernetes.io/docs/tasks/configure-pod-container/configmap/)安装为文件

Kubernetes [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 还允许我们加载一个文件作为 ConfigMap 属性。这给了我们一个有趣的选择，通过 Kubernetes [ConfigMaps](https://developers.redhat.com/search?t=ConfigMap) 加载 Spring Boot *应用程序。*

为了能够通过 [配置映射](https://developers.redhat.com/search?t=ConfigMap)加载 *application.properties，我们需要挂载[配置映射](https://developers.redhat.com/search?t=ConfigMap)作为 Spring Boot 应用程序容器内的卷。*

#### 更新应用程序.属性

```
greeter.message=%s %s! Spring Boot application.properties has been mounted as volume on Kubernetes!
```

#### 从文件创建配置映射

```
kubectl create configmap spring-app-config --from-file=src/main/resources/application.properties
```

上面的命令将创建一个名为 **spring-app-config** 的配置映射，并将 application.properties 文件存储为属性之一。

`kubectl get configmap spring-app-config -o yaml`的样本输出如下所示。

```
apiVersion: v1
data:
  application.properties: greeter.message=%s %s! Spring Boot application.properties has been mounted as volume on Kubernetes!
    on Kubernetes!
kind: ConfigMap
metadata:
  creationTimestamp: 2017-09-19T04:45:27Z
  name: spring-app-config
  namespace: default
  resourceVersion: "53471"
  selfLink: /api/v1/namespaces/default/configmaps/spring-app-config
  uid: 5bac774a-9cf5-11e7-9b8d-080027da6995
```

#### 修改欢迎控制器

```
package com.redhat.developers;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class GreeterController {

    @Value("${greeter.message}")
    private String greeterMessageFormat; 

    @GetMapping("/greet/{user}")
    public String greet(@PathVariable("user") String user) {
        String prefix = System.getenv().getOrDefault("GREETING_PREFIX", "Hi");
        log.info("Prefix :{} and User:{}", prefix, user);
        if (prefix == null) {
            prefix = "Hello!";
        }

        return String.format(greeterMessageFormat, prefix, user);
    }
}
```

#### 更新片段部署. yaml

更新 **deployment.yaml** 以添加卷挂载，这将允许我们在 **/deployments/config** 下挂载**应用程序.属性**。

```
spec:
  template:
    spec:
      containers:
        - env:
          - name: GREETING_PREFIX
            valueFrom:
             configMapKeyRef:
                name: spring-boot-configmaps-demo
                key: greeter.prefix
          volumeMounts:
          - name: application-config 
            mountPath: "/deployments/config" 
            readOnly: true
      volumes:
      - name: application-config
        configMap:
          name: spring-app-config 
          items:
          - key: application.properties 
            path: application.properties
```

让我们像前面那样[部署](#deploy-application)和[访问](#accessing-application)应用程序，但是这次响应将使用来自[配置映射](https://developers.redhat.com/search?t=ConfigMap)的 application.properties。

## Spring Boot 和库伯内特系列

[如何在 Kubernetes 上配置 Spring Boot 应用](https://developers.redhat.com/blog/2017/10/02/configuring-spring-boot-application-kubernetes/)

第一部分:[使用 ConfigMap 在 Kubernetes 上配置 Spring Boot](https://developers.redhat.com/blog/2017/10/03/configuring-spring-boot-kubernetes-configmap/)

第二部分:[配置 Spring Boot·库伯内特的秘密](https://developers.redhat.com/blog/2017/10/04/configuring-spring-boot-kubernetes-secrets/)

*Last updated: November 4, 2021*