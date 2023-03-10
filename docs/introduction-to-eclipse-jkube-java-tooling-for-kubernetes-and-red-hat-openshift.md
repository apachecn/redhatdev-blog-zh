# Eclipse JKube 简介:Kubernetes 和 Red Hat OpenShift 的 Java 工具

> 原文：<https://developers.redhat.com/blog/2020/01/28/introduction-to-eclipse-jkube-java-tooling-for-kubernetes-and-red-hat-openshift>

作为 Java 开发人员，我们经常忙于优化应用程序的内存、速度等。近年来，将我们的应用程序封装到称为容器的轻量级独立单元中已经成为一种趋势，几乎每个企业都在试图将其基础设施转移到 Docker 和 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 这样的容器技术上。

Kubernetes 是一个用于自动化部署、扩展和管理容器化应用程序的开源系统，但是它有一个陡峭的学习曲线，一个没有 DevOps 背景的应用程序开发人员可能会觉得这个系统有点难以应付。在本文中，我将讨论在将您的 Maven 应用程序部署到[Kubernetes](https://kubernetes.io/)/[Red Hat open shift](https://developers.redhat.com/openshift)时可以提供帮助的工具。

## **背景:日蚀 JKube**

这个项目不是从零开始的。这只是一个重构和重塑版本的 [Fabric8 Maven 插件](https://github.com/fabric8io/fabric8-maven-plugin)，这是一个在 [Fabric8](http://fabric8.io) 生态系统中使用的 Maven 插件。虽然 Fabric8 项目受到了开源社区中许多人的喜欢和赞赏，但由于不幸的原因，它无法获得成功，Fabric8 作为 Kubernetes 之上的集成开发平台的想法也就破灭了。虽然主项目已经归档，但是仍然有活跃的库被社区使用，比如 [Fabric8 Docker Maven 插件](https://github.com/fabric8io/docker-maven-plugin)、 [Fabric8 Kubernetes 客户端](https://github.com/fabric8io/kubernetes-client)，当然还有 Fabric8 Maven 插件。

作为 Fabric8 Maven 插件的维护者，我们开始将 Fabric8 生态系统相关部分从插件中分离出来，以制作一个通用的 Kubernetes/OpenShift 插件。我们也觉得有必要重塑品牌，因为大多数人都不清楚这个插件是否与 Fabric8 有关。因此，我们决定重新命名它，幸运的是，来自 Eclipse foundation 的人找到我们，接受我们的项目。现在，这个项目被重新命名为 Eclipse JKube，并且可以在 GitHub 上的 Eclipse Foundation repos 中找到。

Eclipse JKube 可以看作是 Fabric8 Maven 插件的转世。它包含了这个插件的好的部分，并且用它提供的工具提供了一个干净流畅的工作流程。我们将这个插件重构为三个组件:

*   [JKube 套件](https://github.com/eclipse/jkube/tree/master/jkube-kit)
*   [【立方 Maven 插件】](https://github.com/eclipse/jkube/tree/master/kubernetes-maven-plugin)
*   [open shift Maven 插件](https://github.com/eclipse/jkube/tree/master/openshift-maven-plugin)

JKube 工具包包含构建 Docker 映像、生成 Kubernetes/OpenShift 清单以及将它们应用到 Kubernetes/OpenShift 集群的核心逻辑。插件使用这个库进行操作。未来，我们还计划增加对 Gradle 插件的支持。

## 例子

现在，让我们来看看 Eclipse JKube 的运行情况。为了演示，我将使用 [Eclipse Kubernetes Maven 插件](https://github.com/rohanKanojia/eclipse-jkube-demo-project)在 Kubernetes 上部署一个简单的 Spring Boot 项目。让我们来看一下这个过程:

1.  将 [Kubernetes Maven 插件](https://search.maven.org/search?q=g:%22org.eclipse.jkube%22%20AND%20a:%22kubernetes-maven-plugin%22)作为依赖项添加到您的`pom.xml`文件中，如下所示:

```
<plugin>
    <groupId>org.eclipse.jkube</groupId>
    <artifactId>kubernetes-maven-plugin</artifactId>
    <version>${jkube.version}</version>
</plugin>
```

2.  Build your Docker images. The Eclipse JKube Kubernetes Maven plugin offers a zero-config mode, in which it builds your Docker image with opinionated defaults. Right now I'm just customizing the name of the image created with the `jkube.generator.name` property to include my Docker Hub username. However, you can also customize it by providing an image configuration in the plugin configuration.Here is an example with my DockerHub username:

    ```
    <jkube.generator.name>docker.io/rohankanojia/random-generator:${project.version}</jkube.generator.name>
    ```

    好了，现在您可以使用 Eclipse JKube 来封装您的应用程序了。为了构建 Docker 映像，您只需运行以下命令:

```
eclipse-jkube-demo-project : $ mvn k8s:build
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< meetup:random-generator >-----------------------
[INFO] Building random-generator 0.0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- kubernetes-maven-plugin:1.0.1:build (default-cli) @ random-generator ---
[INFO] k8s: Running in Kubernetes mode
[INFO] k8s: Building Docker image in Kubernetes mode
[INFO] k8s: Running generator spring-boot
[INFO] k8s: spring-boot: Using Docker image quay.io/jkube/jkube-java-binary-s2i:0.0.8 as base / builder
[INFO] k8s: Pulling from jkube/jkube-java-binary-s2i
0fd3b5213a9b: Pulling fs layer 
aebb8c556853: Pulling fs layer 
0fd3b5213a9b: Downloading [>                                                  ]  539.5kB/54.37MB
595bd04e186b: Downloading [>                                                  ]  2.158MB/133.7MB
0fd3b5213a9b: Downloading [>                                                  ]   1.08MB/54.37MB
595bd04e186b: Downloading [=>                                                 ]  5.349MB/133.7MB
0fd3b5213a9b: Downloading [=>                                                 ]  1.621MB/54.37MB
595bd04e186b: Downloading [===>                                               ]  8.572MB/133.7MB
0fd3b5213a9b: Downloading [=>                                                 ]  2.162MB/54.37MB
0fd3b5213a9b: Downloading [==>                                                ]  2.702MB/54.37MB
0fd3b5213a9b: Downloading [==>                                                ]  3.243MB/54.37MB
0fd3b5213a9b: Downloading [===>                                               ]  3.784MB/54.37MB
595bd04e186b: Downloading [=====>                                             ]  14.43MB/133.7MB
0fd3b5213a9b: Downloading [===>                                               ]  4.324MB/54.37MB
0fd3b5213a9b: Downloading [====>                                              ]  5.406MB/54.37MB
0fd3b5213a9b: Downloading [=====>                                             ]  6.487MB/54.37MB
0fd3b5213a9b: Downloading [======>                                            ]  7.568MB/54.37MB
595bd04e186b: Downloading [=======>                                           ]  19.79MB/133.7MB
0fd3b5213a9b: Downloading [=======>                                           ]   8.65MB/54.37MB
0fd3b5213a9b: Downloading [========>                                          ]  9.731MB/54.37MB
0fd3b5213a9b: Downloading [=========>                                         ]  10.27MB/54.37MB
0fd3b5213a9b: Downloading [=========>                                         ]  10.81MB/54.37MB
0fd3b5213a9b: Pull complete 
aebb8c556853: Pull complete 
595bd04e186b: Pull complete 
[INFO] k8s: Digest: sha256:69cacf4092e7ac1765395798d42c16efe2bf88e71472eaffbd2764d0ae95a8fe
[INFO] k8s: Status: Downloaded newer image for quay.io/jkube/jkube-java-binary-s2i:0.0.8
[INFO] k8s: Pulled quay.io/jkube/jkube-java-binary-s2i:0.0.8 in 27 seconds 
[INFO] k8s: [rohankanojia/random-generator:0.0.1] "spring-boot": Created docker-build.tar in 141 milliseconds
[INFO] k8s: [rohankanojia/random-generator:0.0.1] "spring-boot": Built image sha256:a3c06
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  33.047 s
[INFO] Finished at: 2020-10-13T23:52:24+05:30
[INFO] ------------------------------------------------------------------------
eclipse-jkube-demo-project : $ docker images | grep random-generator
rohankanojia/random-generator                 0.0.1               a3c06251eed7        9 seconds ago       528 MB

```

3.  生成您的 Kubernetes 资源清单。Eclipse JKube 插件有一个强大的可配置资源生成机制，允许它们在零配置模式下生成 Kubernetes 资源。这个特性也可以使用 XML 配置或者通过将定制的资源片段放在`src/main/jkube`目录中来配置。结果与最终生成的资源片段合并。为了生成资源，请运行以下命令:

```
eclipse-jkube-demo-project : $ mvn k8s:resource
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< meetup:random-generator >-----------------------
[INFO] Building random-generator 0.0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- kubernetes-maven-plugin:1.0.1:resource (default-cli) @ random-generator ---
[INFO] k8s: Running generator spring-boot
[INFO] k8s: spring-boot: Using Docker image quay.io/jkube/jkube-java-binary-s2i:0.0.8 as base / builder
[INFO] k8s: Using resource templates from /home/rohaan/work/repos/eclipse-jkube-demo-project/src/main/jkube
[INFO] k8s: jkube-controller: Adding a default Deployment
[INFO] k8s: jkube-service: Adding a default service 'random-generator' with ports [8080]
[INFO] k8s: jkube-healthcheck-spring-boot: Adding readiness probe on port 8080, path='/actuator/health', scheme='HTTP', with initial delay 10 seconds
[INFO] k8s: jkube-healthcheck-spring-boot: Adding liveness probe on port 8080, path='/actuator/health', scheme='HTTP', with initial delay 180 seconds
[INFO] k8s: jkube-revision-history: Adding revision history limit to 2
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.940 s
[INFO] Finished at: 2020-10-13T23:53:39+05:30
[INFO] ------------------------------------------------------------------------
eclipse-jkube-demo-project : $ ls target/classes/META-INF/jkube/
kubernetes  kubernetes.yml
eclipse-jkube-demo-project : $ ls target/classes/META-INF/jkube/kubernetes
random-generator-deployment.yml  random-generator-service.yml
eclipse-jkube-demo-project : $ 

```

4.  将生成的 Kubernetes 资源应用到 Kubernetes 集群。为了将资源应用到该集群上，请运行以下命令之一(结果显示在下面的列表中):

```
$ mvn k8s:apply
```

或者:

```
$ mvn k8s:deploy

```

```
eclipse-jkube-demo-project : $ mvn k8s:apply
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< meetup:random-generator >-----------------------
[INFO] Building random-generator 0.0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- kubernetes-maven-plugin:1.0.1:apply (default-cli) @ random-generator ---
[INFO] k8s: Using Kubernetes at https://192.168.39.129:8443/ in namespace default with manifest /home/rohaan/work/repos/eclipse-jkube-demo-project/target/classes/META-INF/jkube/kubernetes.yml 
[INFO] k8s: Using namespace: default
[INFO] k8s: Creating a Service from kubernetes.yml namespace default name random-generator
[INFO] k8s: Created Service: target/jkube/applyJson/default/service-random-generator.json
[INFO] k8s: Creating a Deployment from kubernetes.yml namespace default name random-generator
[INFO] k8s: Created Deployment: target/jkube/applyJson/default/deployment-random-generator.json
[INFO] k8s: HINT: Use the command `kubectl get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.282 s
[INFO] Finished at: 2020-10-13T23:55:10+05:30
[INFO] ------------------------------------------------------------------------
eclipse-jkube-demo-project : $ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
random-generator-8697c5d7d6-k9s7d   0/1     Running   0          9s
eclipse-jkube-demo-project : $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/random-generator-8697c5d7d6-k9s7d   0/1     Running   0          12s

NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP          4h3m
service/random-generator   NodePort    10.106.53.34   <none>        8080:32404/TCP   12s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-generator   0/1     1            0           12s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/random-generator-8697c5d7d6   1         1         0       12s
eclipse-jkube-demo-project : $ kubectl get svc
NAME               TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes         ClusterIP   10.96.0.1      <none>        443/TCP          4h4m
random-generator   NodePort    10.106.53.34   <none>        8080:32404/TCP   36s
eclipse-jkube-demo-project : $ curl `minikube ip`:32404/random | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    45    0    45    0     0   2500      0 --:--:-- --:--:-- --:--:--  2500
{
  "id": "fb4f1cdb-cdd2-4604-ada3-35c27ebd733b"
}

```

5.  从 Kubernetes 取消部署您的 Maven 应用程序。我们还有一个清理目标，即删除部署阶段创建的所有资源。要使用此功能，请运行以下命令(结果如下所示):

```
$ mvn k8s:undeploy

```

```
eclipse-jkube-demo-project : $ kubectl get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/random-generator-8697c5d7d6-mxv7g   1/1     Running   0          45s

NAME                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
service/kubernetes         ClusterIP   10.96.0.1     <none>        443/TCP          4h7m
service/random-generator   NodePort    10.107.3.32   <none>        8080:32287/TCP   45s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/random-generator   1/1     1            1           45s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/random-generator-8697c5d7d6   1         1         1       45s
eclipse-jkube-demo-project : $ mvn k8s:undeploy
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< meetup:random-generator >-----------------------
[INFO] Building random-generator 0.0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- kubernetes-maven-plugin:1.0.1:undeploy (default-cli) @ random-generator ---
[INFO] k8s: Deleting resource Deployment default/random-generator
[INFO] k8s: Deleting resource Service default/random-generator
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.662 s
[INFO] Finished at: 2020-10-13T23:58:39+05:30
[INFO] ------------------------------------------------------------------------
eclipse-jkube-demo-project : $ kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4h7m
eclipse-jkube-demo-project : $ 

```

6.  在 Kubernetes 内部调试 Java 应用程序。除了这些目标，我们还有一个远程调试的目标。假设您在 Kubernetes 中运行的应用程序中发现了一个 bug，您想调试它的行为。您可以简单地运行我们的调试目标，它为调试进行端口转发:

```
eclipse-jkube-demo-project : $ mvn k8s:debug
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< meetup:random-generator >-----------------------
[INFO] Building random-generator 0.0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- kubernetes-maven-plugin:1.0.1:debug (default-cli) @ random-generator ---
[INFO] k8s: Using Kubernetes at https://192.168.39.129:8443/ in namespace default with manifest /home/rohaan/work/repos/eclipse-jkube-demo-project/target/classes/META-INF/jkube/kubernetes.yml 
[INFO] k8s: Using namespace: default
[INFO] k8s: Updating Service from kubernetes.yml
[INFO] k8s: Updated Service: target/jkube/applyJson/default/service-random-generator-5.json
[INFO] k8s: Enabling debug on Deployment random-generator
[INFO] k8s: Waiting for debug pod with selector LabelSelector(matchExpressions=[], matchLabels={app=random-generator, provider=jkube, group=meetup}, additionalProperties={}) and environment variables {JAVA_DEBUG_SUSPEND=false, JAVA_ENABLE_DEBUG=true}
[INFO] k8s: Port forwarding to port 5005 on pod random-generator-69f9947655-ws4pk using command /home/rohaan/.local/bin/kubectl

```

7.  配置您的 IDE，以便连接到这个开放端口进行调试，如图 1 所示:

[![](img/8a58f2f3dec5549e358cced054d9ba9a.png "Figure 1: Configuring your IDE to debug application inside Kubernetes.")](/sites/default/files/blog/2020/01/Screenshot-from-2020-10-14-00-21-03.png)

Figure 1: Configuring your IDE to debug application inside Kubernetes.

8.  在应用程序代码中设置断点，并点击应用程序端点。我们可以看到断点在 IDE 中被点击，如图 2 所示:

[![Figure 2: Configuring your IDE to debug an application running inside Kubernetes.](img/61d54093e6e06a14a3d01ad2d17ca340.png "Screenshot from 2020-10-14 00-25-58")](/sites/default/files/blog/2020/01/Screenshot-from-2020-10-14-00-25-58.png)Figure 2: Configuring your IDE to debug an application running inside Kubernetes.

Figure 2: Configuring your IDE to debug an application running inside Kubernetes.

## 部署到 Red Hat OpenShift:

您可以使用 [Eclipse JKube 的](https://github.com/eclipse/jkube) [OpenShift Maven 插件将相同的应用程序部署到](https://search.maven.org/search?q=g:%22org.eclipse.jkube%22%20AND%20a:%22openshift-maven-plugin%22) [Red Hat OpenShift](https://www.openshift.com/) 。你可以这样做:

1.  在您的`pom.xml`的`<plugins>`部分添加 OpenShift Maven 插件:

```
<plugin>
    <groupId>org.eclipse.jkube</groupId>
    <artifactId>openshift-maven-plugin</artifactId>
    <version>${jkube.version}</version>
</plugin>
```

2.  登录您的 OpenShift 集群:

```
$ oc login https://api.example.openshift.com --token=some-token
Logged into "https://api.example.openshift.com:443" as "rohanKanojia" using the token provided.

You have one project on this server: "rokumar"

Using project "rokumar".
```

```
-        <jkube.generator.name>docker.io/rohankanojia/random-generator:${project.version}</jkube.generator.name>

```

删除该属性后，您可以继续执行部署到 Red Hat OpenShift 的常规流程:

```
eclipse-jkube-demo-project : $ mvn oc:build oc:resource oc:apply
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------------< meetup:random-generator >-----------------------
[INFO] Building random-generator 0.0.1
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- openshift-maven-plugin:1.0.1:build (default-cli) @ random-generator ---
[INFO] oc: Using OpenShift build with strategy S2I
[INFO] oc: Running in OpenShift mode
[INFO] oc: Running generator spring-boot
[INFO] oc: spring-boot: Using Docker image quay.io/jkube/jkube-java-binary-s2i:0.0.8 as base / builder
[INFO] oc: [random-generator:0.0.1] "spring-boot": Created docker source tar /home/rohaan/work/repos/eclipse-jkube-demo-project/target/docker/random-generator/0.0.1/tmp/docker-build.tar
[INFO] oc: Adding to Secret pullsecret-jkube
[INFO] oc: Using Secret pullsecret-jkube
[INFO] oc: Creating BuildServiceConfig random-generator-s2i for Source build
[INFO] oc: Creating ImageStream random-generator
[INFO] oc: Starting Build random-generator-s2i
[INFO] oc: Waiting for build random-generator-s2i-1 to complete...
[INFO] oc: Using quay.io/jkube/jkube-java-binary-s2i:0.0.8 as the s2i builder image
[INFO] oc: INFO S2I source build with plain binaries detected
[INFO] oc: INFO S2I binary build from fabric8-maven-plugin detected
[INFO] oc: INFO Copying binaries from /tmp/src/deployments to /deployments ...
[INFO] oc: random-generator-0.0.1.jar
[INFO] oc: INFO Copying deployments from deployments to /deployments...
[INFO] oc: '/tmp/src/deployments/random-generator-0.0.1.jar' -> '/deployments/random-generator-0.0.1.jar'
[INFO] oc: INFO Cleaning up source directory (/tmp/src)
[INFO] oc: 
[INFO] oc: Pushing image 172.30.39.149:5000/rokumar/random-generator:0.0.1 ...
[INFO] oc: Pushed 3/4 layers, 95% complete
[INFO] oc: Pushed 4/4 layers, 100% complete
[INFO] oc: Push successful
[INFO] oc: Build random-generator-s2i-1 in status Complete
[INFO] oc: Found tag on ImageStream random-generator tag: sha256:4c71b89b5345db80ab4802e127085ac45bd1410e6e91439fad956fe12200b93c
[INFO] oc: ImageStream random-generator written to /home/rohaan/work/repos/eclipse-jkube-demo-project/target/random-generator-is.yml
[INFO] 
[INFO] --- openshift-maven-plugin:1.0.1:resource (default-cli) @ random-generator ---
[INFO] oc: Using docker image name of namespace: rokumar
[INFO] oc: Running generator spring-boot
[INFO] oc: spring-boot: Using Docker image quay.io/jkube/jkube-java-binary-s2i:0.0.8 as base / builder
[INFO] oc: Using resource templates from /home/rohaan/work/repos/eclipse-jkube-demo-project/src/main/jkube
[INFO] oc: jkube-controller: Adding a default DeploymentConfig
[INFO] oc: jkube-service: Adding a default service 'random-generator' with ports [8080]
[INFO] oc: jkube-healthcheck-spring-boot: Adding readiness probe on port 8080, path='/actuator/health', scheme='HTTP', with initial delay 10 seconds
[INFO] oc: jkube-healthcheck-spring-boot: Adding liveness probe on port 8080, path='/actuator/health', scheme='HTTP', with initial delay 180 seconds
[INFO] oc: jkube-revision-history: Adding revision history limit to 2
[INFO] 
[INFO] --- openshift-maven-plugin:1.0.1:apply (default-cli) @ random-generator ---
[INFO] oc: Using OpenShift at https://api.rh-idev.openshift.com:443/ in namespace rokumar with manifest /home/rohaan/work/repos/eclipse-jkube-demo-project/target/classes/META-INF/jkube/openshift.yml 
[INFO] oc: OpenShift platform detected
[INFO] oc: Using project: rokumar
[INFO] oc: Creating a Service from openshift.yml namespace rokumar name random-generator
[INFO] oc: Created Service: target/jkube/applyJson/rokumar/service-random-generator.json
[INFO] oc: Creating a DeploymentConfig from openshift.yml namespace rokumar name random-generator
[INFO] oc: Created DeploymentConfig: target/jkube/applyJson/rokumar/deploymentconfig-random-generator.json
[INFO] oc: Creating Route rokumar:random-generator host: null
[INFO] oc: HINT: Use the command `oc get pods -w` to watch your pods start up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  48.578 s
[INFO] Finished at: 2020-10-14T00:57:20+05:30
[INFO] ------------------------------------------------------------------------
eclipse-jkube-demo-project : $ oc get pods
NAME                           READY     STATUS      RESTARTS   AGE
random-generator-1-vtdzs       1/1       Running     0          4m
random-generator-s2i-1-build   0/1       Completed   0          5m
eclipse-jkube-demo-project : $ oc get routes
NAME               HOST/PORT                                                 PATH      SERVICES           PORT      TERMINATION   WILDCARD
random-generator   random-generator-rokumar.b6ff.rh-idev.openshiftapps.com             random-generator   8080                    None
eclipse-jkube-demo-project : $ curl random-generator-rokumar.b6ff.rh-idev.openshiftapps.com/random | jq .
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    45    0    45    0     0     62      0 --:--:-- --:--:-- --:--:--    62
{
  "id": "c656151e-c48a-4331-b265-859a16209b14"
}
```

有了这个结果，我就结束了这篇文章。我们确实有更多的在我们的管道中，所以请继续关注新的更新。

## 结论:

无论您已经在使用 Eclipse JKube 还是只是好奇，请不要羞于加入我们的欢迎社区:

*Last updated: October 28, 2020*