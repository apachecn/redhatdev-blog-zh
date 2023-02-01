# Fabric8 中的新增功能 Kubernetes Java 客户端 4.12.0

> 原文：<https://developers.redhat.com/blog/2020/10/30/whats-new-in-fabric8-kubernetes-java-client-4-12-0>

最近的[fabric 8 Kubernetes Java client](https://github.com/fabric8io/kubernetes-client)[4 . 12 . 0](https://github.com/fabric8io/kubernetes-client/releases/tag/v4.12.0)版本包括许多新功能和错误修复。这篇文章介绍了我们在 [4.11.0](https://github.com/fabric8io/kubernetes-client/releases/tag/v4.11.0) 和 [4.12.0](https://github.com/fabric8io/kubernetes-client/releases/tag/v4.12.0) 版本之间添加的主要特性。

我将向您展示如何在 Fabric8 Tekton 客户端中开始使用新的`VolumeSnapshot`扩展、`CertificateSigningRequests`和 Tekton 触发器(仅举几例)。我还将指出几个打破与旧版本向后兼容性的小变化。了解这些变化将有助于你在升级到最新版本的 Fabric8 的 [Java](https://developers.redhat.com/topics/enterprise-java/) 客户端用于 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 或 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 时避免出现问题。

## 如何获得新的 Fabric8 Java 客户端

你可以在 [Maven Central](https://search.maven.org/artifact/io.fabric8/kubernetes-client/4.12.0/jar) 上找到最新的 Fabric8 Java 客户端版本。要开始使用新的 Java 客户端，将其作为一个依赖项添加到您的 Maven `pom.xml`中。对于 Kubernetes，依赖性是:

```
<dependency>
  <groupId>io.fabric8</groupId>
  <artifactId>kubernetes-client</artifactId>
  <version>4.12.0</version>
</dependency>

```

对于 OpenShift，它是:

```
<dependency>
  <groupId>io.fabric8</groupId>
  <artifactId>openshift-client</artifactId>
  <version>4.12.0</version>
</dependency>

```

## 此版本中的重大变化

我们已经为这个版本移动了几个类，所以升级到 Fabric8 Kubernetes Java 客户端的新版本可能不会完全顺利。这些变化如下:

*   我们把`CustomResourceDefinition`移到了`io.fabric8.kubernetes.api.model.apiextensions.v1`和`io.fabric8.kubernetes.api.model.apiextensions.v1beta1`。
*   我们把`SubjectAccessReview`、`SelfSubjectAccessReview`、`LocalSubjectAccessReview`和`SelfSubjectRulesReview`移到了`io.fabric8.kubernetes.api.model.authorization.v1`和`io.fabric8.kubernetes.api.model.authorization.v1beta1`。
*   `io.fabric8.tekton.pipeline.v1beta1.WorkspacePipelineDeclaration`现在是`io.fabric8.tekton.pipeline.v1beta1.PipelineWorkspaceDeclaration`。
*   我们引入了一个新的接口`WatchAndWaitable`，由`WatchListDeletable`和其他接口使用。如果您使用的是 Fabric8 Kubernetes Java 客户端的[特定于域的语言(DSL)](https://javadoc.io/doc/io.fabric8/kubernetes-client/latest/index.html) ，这个变化应该不会影响到您。

## 新的卷快照扩展

你可能知道用于 [Knative](https://developers.redhat.com/topics/serverless-architecture) 、 [Tekton](https://developers.redhat.com/topics/ci-cd) 、 [Istio](https://developers.redhat.com/topics/service-mesh) 和服务目录的 [Fabric8 Kubernetes Java 客户端扩展](https://github.com/fabric8io/kubernetes-client/tree/master/extensions)。在这个版本中，我们添加了一个新的[容器存储接口](https://github.com/container-storage-interface/spec) (CSI) `VolumeSnapshot`扩展。`VolumeSnapshot` s 在`snapshot.storage.k8s.io/v1beta1`目录中。要开始使用新的扩展，请向您的 Maven `pom.xml`添加以下依赖项:

```
<dependency>
  <groupId>io.fabric8</groupId>
  <artifactId>volumesnapshot-client</artifactId>
  <version>4.12.0</version>
</dependency>

```

一旦添加了依赖项，就可以开始使用`VolumeSnapshotClient`。下面是一个如何创建`VolumeSnapshot`的例子:

```
try (VolumeSnapshotClient client = new DefaultVolumeSnapshotClient()) {
      System.out.println("Creating a volume snapshot");
      client.volumeSnapshots().inNamespace("default").createNew()
        .withNewMetadata()
        .withName("my-snapshot")
        .endMetadata()
        .withNewSpec()
        .withNewSource()
        .withNewPersistentVolumeClaimName("my-pvc")
        .endSource()
        .endSpec()
        .done();
    }

```

## 用`client.run()`旋转单个吊舱

就像您使用`kubectl run`一样，您可以使用 Fabric8 Kubernetes Java 客户端快速启动 pod。您只需提供姓名和图像:

```
try (KubernetesClient client = new DefaultKubernetesClient()) {
    client.run().inNamespace("default").withName("hello-openshift")
            .withImage("openshift/hello-openshift:latest")
            .done();
}

```

## 身份验证 API 支持

一个新的身份验证 API 允许您使用 Fabric8 Kubernetes Java 客户端来查询 Kubernetes 集群。你应该能够使用 API 进行所有等同于`kubectl auth can-i`的操作。这里有一个例子:

```
try (KubernetesClient client = new DefaultKubernetesClient()) {
    SelfSubjectAccessReview ssar = new SelfSubjectAccessReviewBuilder()
            .withNewSpec()
            .withNewResourceAttributes()
            .withGroup("apps")
            .withResource("deployments")
            .withVerb("create")
            .withNamespace("dev")
            .endResourceAttributes()
            .endSpec()
            .build();

    ssar = client.authorization().v1().selfSubjectAccessReview().create(ssar);

    System.out.println("Allowed: "+  ssar.getStatus().getAllowed());
}

```

## OpenShift 4 资源

Fabric8 Kubernetes Java 客户端现在在其 OpenShift 模型中支持所有新的 OpenShift 4 资源。在`operators.coreos.com`、`operators.openshift.io`、`console.openshift.io`和`monitoring.coreos.com`中添加的附加资源也可在 OpenShift 模型中获得。下面是一个使用`PrometheusRule`来监控`Prometheus`实例的例子:

```
try (OpenShiftClient client = new DefaultOpenShiftClient()) {
    PrometheusRule prometheusRule = new PrometheusRuleBuilder()
            .withNewMetadata().withName("foo").endMetadata()
            .withNewSpec()
            .addNewGroup()
            .withName("./example-rules")
            .addNewRule()
            .withAlert("ExampleAlert")
            .withNewExpr().withStrVal("vector(1)").endExpr()
            .endRule()
            .endGroup()
            .endSpec()
            .build();

    client.monitoring().prometheusRules().inNamespace("rokumar").createOrReplace(prometheusRule);
    System.out.println("Created");

    PrometheusRuleList prometheusRuleList = client.monitoring().prometheusRules().inNamespace("rokumar").list();
    System.out.println(prometheusRuleList.getItems().size() + " items found");
}

```

## 证书签名请求

我们在主`KubernetesClient`界面中添加了一个新的入口点`certificateSigningRequests()`。这意味着您可以在所有使用 Fabric8 开发的应用程序中使用[证书设计请求资源](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/):

```
try (KubernetesClient client = new DefaultKubernetesClient()) {

    CertificateSigningRequest csr = new CertificateSigningRequestBuilder()
            .withNewMetadata().withName("test-k8s-csr").endMetadata()
            .withNewSpec()
            .addNewGroup("system:authenticated")
            .withRequest("<your-req>")
            .addNewUsage("client auth")
            .endSpec()
            .build();
    client.certificateSigningRequests().create(csr);
}

```

## 自定义资源定义

我们已经将`apiextensions/v1` `CustomResourceDefinition` (CRD)移到了`io.fabric8.kubernetes.api.model.apiextensions.v1beta1`和`io.fabric8.kubernetes.api.model.apiextensions.v1`包中。你现在可以像这样使用`apiextensions()`中的`CustomResourceDefinition`对象:

```
try (KubernetesClient client = new DefaultKubernetesClient()) {
    client.apiextensions().v1()
            .customResourceDefinitions()
            .list()
            .getItems().forEach(crd -> System.out.println(crd.getMetadata().getName()));
}

```

## 创建引导项目模板

我们提供了一种新的内置方式来创建一个包含您需要的所有角色绑定的项目。它的工作原理类似于 OpenShift 的`oc adm create-bootstrap-project-template`命令。指定模板在 DSL 方法中需要的参数。该方法然后为您创建`Project`和相关的`RoleBindings`:

```
try (OpenShiftClient client = new DefaultOpenShiftClient()) {
    client.projects().createProjectAndRoleBindings("default", "Rohan Kumar", "default", "developer", "developer");
}

```

## Tekton 型号 0.15.1

我们已经将 Tekton 模型更新到版本 0.15.1，以便您可以利用 Tekton 的所有最新上游功能和增强功能。这个例子创建了一个简单的`Task`和`TaskRun`来响应 pod 中的“hello world”。我们用 [Fabric8 TektonClient](https://search.maven.org/artifact/io.fabric8/tekton-client/4.12.0/bundle) 代替`YAML`:

```
try (TektonClient tkn = new DefaultTektonClient()) {
    // Create Task
    tkn.v1beta1().tasks().inNamespace(NAMESPACE).createOrReplaceWithNew()
            .withNewMetadata().withName("echo-hello-world").endMetadata()
            .withNewSpec()
            .addNewStep()
            .withName("echo")
            .withImage("alpine:3.12")
            .withCommand("echo")
            .withArgs("Hello World")
            .endStep()
            .endSpec()
            .done();

    // Create TaskRun
    tkn.v1beta1().taskRuns().inNamespace(NAMESPACE).createOrReplaceWithNew()
            .withNewMetadata().withName("echo-hello-world-task-run").endMetadata()
            .withNewSpec()
            .withNewTaskRef()
            .withName("echo-hello-world")
            .endTaskRef()
            .endSpec()
            .done();
}

```

当您运行这段代码时，您将看到正在创建的`Task`和`TaskRun`。反过来，`TaskRun`创建一个 pod，打印“Hello World”消息:

```
tekton-java-client-demo : $ tkn taskrun list
NAME                        STARTED         DURATION     STATUS
echo-hello-world-task-run   2 minutes ago   19 seconds   Succeeded
tekton-java-client-demo : $ kubectl get pods
NAME                                  READY   STATUS      RESTARTS   AGE
echo-hello-world-task-run-pod-4gczw   0/1     Completed   0          2m17s
tekton-java-client-demo : $ kubectl logs pod/echo-hello-world-task-run-pod-4gczw
Hello World

```

## Fabric8 Tekton 客户端中的 Tekton 触发器

[Fabric8 Tekton 客户端](https://search.maven.org/artifact/io.fabric8/tekton-client/4.12.0/bundle)和[型号](https://search.maven.org/artifact/io.fabric8/tekton-model-v1beta1/4.12.0/bundle)现在支持 [Tekton 触发器](https://github.com/tektoncd/triggers)。您可以使用触发器来自动创建 Tekton 管道。您所要做的就是在 Tekton 连续部署(CD)管道中嵌入您的触发器。下面是一个使用 Fabric8 Tekton 客户端创建 [Tekton 触发器模板](https://github.com/tektoncd/triggers/blob/3a2ecf8b3ef143a81d6908d055bef4f3251ed01d/examples/triggertemplates/triggertemplate.yaml#L1-L38)的示例:

```
try (TektonClient tkn = new DefaultTektonClient()) {
    tkn.v1alpha1().triggerTemplates().inNamespace(NAMESPACE).createOrReplaceWithNew()
            .withNewMetadata().withName("pipeline-template").endMetadata()
            .withNewSpec()
                .addNewParam()
                    .withName("gitrepositoryurl")
                    .withDescription("The git repository url")
                .endParam()
                .addNewParam()
                    .withName("gitrevision")
                    .withDescription("The git revision")
                .endParam()
                .addNewParam()
                    .withName("message")
                    .withDescription("The message to print")
                    .withDefault("This is default message")
                .endParam()
                .addNewParam()
                    .withName("contenttype")
                    .withDescription(" The Content-Type of the event")
                .endParam()
            .withResourcetemplates(Collections.singletonList(new PipelineRunBuilder()
                    .withNewMetadata().withGenerateName("simple-pipeline-run-").endMetadata()
                    .withNewSpec()
                        .withNewPipelineRef().withName("simple-pipeline").endPipelineRef()
                        .addNewParam()
                            .withName("message")
                            .withValue(new ArrayOrString("$(tt.params.message)"))
                        .endParam()
                        .addNewParam()
                            .withName("contenttype")
                            .withValue(new ArrayOrString("$(tt.params.contenttype)"))
                        .endParam()
                        .addNewResource()
                            .withName("git-source")
                            .withNewResourceSpec()
                                .withType("git")
                                .addNewParam()
                                .withName("revision")
                                .withValue("$(tt.params.gitrevision)")
                                .endParam()
                                .addNewParam()
                                .withName("url")
                                .withValue("$(tt.params.gitrepositoryurl)")
                                .endParam()
                            .endResourceSpec()
                        .endResource()
                    .endSpec()
                    .build()))
            .endSpec()
            .done();
}

```

## 自动刷新 OpenID 连接令牌

如果你的 Kubernetes 提供商使用 [OpenID Connect 令牌](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#openid-connect-tokens)(像 [IBM Cloud](https://www.ibm.com/cloud) )，你不需要担心你的令牌过期。新的 Fabric8 Kubernetes Java 客户端通过联系 OpenID Connect 提供者自动刷新您的令牌，OpenID Connect 提供者在`~/.kube/config`中列出。

## 支持 Knative 0.17.2 和 Knative Eventing Contrib

对于这个版本，我们已经将 [Knative](https://knative.dev/) 模型更新到最新版本。我们还增加了对来自 [Knative Eventing Contrib](https://github.com/knative/eventing-contrib) 的额外资源的新支持，其中涉及[源](https://knative.dev/docs/eventing/sources/)和与 Apache CouchDB、Apache Kafka、亚马逊简单队列服务(AWS SQS)、GitHub、GitLab 等集成的通道实现。

下面是一个使用`KnativeClient`创建`AwsSqsSource`的例子:

```
try (KnativeClient client = new DefaultKnativeClient()) {
    AwsSqsSource awsSqsSource = new AwsSqsSourceBuilder()
            .withNewMetadata().withName("awssqs-sample-source").endMetadata()
            .withNewSpec()
            .withNewAwsCredsSecret("credentials", "aws-credentials", true)
            .withQueueUrl("QUEUE_URL")
            .withSink(new ObjectReferenceBuilder()
                    .withApiVersion("messaging.knative.dev/v1alpha1")
                    .withKind("Channel")
                    .withName("awssqs-test")
                    .build())
            .endSpec()
            .build();
    client.awsSqsSources().inNamespace("default").createOrReplace(awsSqsSource);
}

```

## 参与进来！

参与开发 [Fabric8 Kubernetes Java 客户端](https://github.com/fabric8io/kubernetes-client)有几种方法:

*   创建 [GitHub 问题](https://github.com/fabric8io/kubernetes-client/issues)来让我们知道什么时候功能没有按预期工作。
*   发送 [pull 请求](https://github.com/fabric8io/kubernetes-client/pulls)进行错误修复和增强。
*   在 Fabric8 Kubernetes Java 客户端 [Gitter 频道](https://gitter.im/fabric8io/kubernetes-client)上与我们聊天。
*   在 [Twitter](https://twitter.com/fabric8io/) 上关注我们。