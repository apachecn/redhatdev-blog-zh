# 如何用 3 个步骤使用 Eclipse JKube Maven 插件

> 原文：<https://developers.redhat.com/blog/2020/07/27/deploy-your-java-web-application-into-the-cloud-using-eclipse-jkube>

在我们有 Spring Boot 和类似的框架之前，web 应用容器是部署 Java web 应用的主要需求。我们现在生活在[微服务](https://developers.redhat.com/topics/microservices)的时代，许多 Java 应用都是在 [Quarkus](https://developers.redhat.com/products/quarkus/getting-started) 、 [Thorntail](https://thorntail.io/) 或 Spring Boot 之上开发的。但是一些用例仍然需要老式的 web 应用程序。

在本文中，您将学习如何使用 [Eclipse JKube](https://developers.redhat.com/blog/2020/01/28/introduction-to-eclipse-jkube-java-tooling-for-kubernetes-and-red-hat-openshift/) 将 Java web 应用程序(WAR)部署到 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 或 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 集群中。我将向您展示，只需添加 [Eclipse JKube](https://www.eclipse.org/jkube) Maven 插件，就可以轻松制作一个单一的 Java web 应用程序云原生版。

## 示例 web 应用程序

出于本文的目的，我们将使用一个非常简单的 [Spring Web MVC](https://docs.spring.io/spring-framework/docs/5.2.7.RELEASE/spring-framework-reference/web.html#spring-web) 应用程序。以下示例显示了 Maven 项目的 [pom.xml](https://github.com/marcnuri-demo/eclipse-jkube-webapp/blob/v0.0.0-tomcat/pom.xml) 的最相关部分:

```
<!-- ... -->
<packaging>war</packaging>
<!-- ... -->
<properties>
  <maven.compiler.source>11</maven.compiler.source>
  <maven.compiler.target>11</maven.compiler.target>
  <failOnMissingWebXml>false</failOnMissingWebXml>
  <!-- ... -->
  <jkube.enricher.jkube-service.type>NodePort</jkube.enricher.jkube-service.type>
</properties>
<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${version.spring}</version>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
<build>
  <plugins>
    <plugin>
      <groupId>org.eclipse.jkube</groupId>
      <artifactId>kubernetes-maven-plugin</artifactId>
      <version>${version.jkube}</version>
    </plugin>
    <!-- ... -->
  </plugins>
</build>

```

首先，请注意`packaging`元素，它表示生成的工件应该打包成一个`war`文件，这是 Java web 应用程序的常用格式。

在 properties 部分，注意项目是为 Java 11 配置的。我们需要一个合适的 JDK 来建造这个项目。我们使用 [failOnMissingWebXml](https://maven.apache.org/plugins/maven-war-plugin/war-mojo.html#failOnMissingWebXml) 选项来配置 [maven-war-plugin](https://maven.apache.org/plugins/maven-war-plugin/) ，这样它就不会因为缺少`web.xml`文件而失败。(你很快就会看到我们用什么来代替。)最后，还有一个特定于 JKube 的属性，`jkube.enricher.jkube-service.type`。这个属性配置 JKube 使用`NodePort`作为`spec.type`来创建服务资源清单。

在依赖项部分，我们只找到两个依赖项。`spring-webmvc`依赖关系让我们可以使用 Spring Web MVC 框架。`javax.servlet-api`依赖项为 Java Servlet API 提供了编译时支持，这是由 web 应用程序容器在运行时提供的。

最后，在插件部分，我们已经配置了 Eclipse JKube 依赖项。注意，我们可以使用`kubernetes-maven-plugin`或`openshift-maven-plugin`。插件的选择取决于我们想要瞄准的集群，但是我们只需要其中的一个。

配置简单明了。与典型的老式 Java web 应用程序的唯一不同之处是 Eclipse JKube 插件依赖性。

### 示例项目中的 Java 类

示例项目包含三个 Java 类:`ExampleInitializer`、`ExampleConfiguration`和`ExampleResource`。首先， [ExampleInitializer](https://github.com/marcnuri-demo/eclipse-jkube-webapp/blob/v0.0.0-tomcat/src/main/java/com/marcnuri/demo/jkube/ExampleInitializer.java) 是一个`WebApplicationInitializer`实现。我们用它代替标准的`WEB-INF/web.xml`部署描述符来编程配置 [ServletContext](https://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html?is-external=true) 。

下面的代码展示了`ExampleInitializer`如何让我们注册 Spring 的`DispatcherServlet`,而不需要任何额外的 XML 配置:

```
final AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
context.register(ExampleConfiguration.class);
context.setServletContext(servletContext);
final ServletRegistration.Dynamic dsr = servletContext.addServlet("dispatcher", new DispatcherServlet(context));
dsr.setLoadOnStartup(1);
dsr.addMapping("/");

```

[ExampleConfiguration](https://github.com/marcnuri-demo/eclipse-jkube-webapp/blob/v0.0.0-tomcat/src/main/java/com/marcnuri/demo/jkube/ExampleConfiguration.java) 类是支持 Spring MVC 的特定于 Spring 的配置。

最后，[示例 Resource](https://github.com/marcnuri-demo/eclipse-jkube-webapp/blob/v0.0.0-tomcat/src/main/java/com/marcnuri/demo/jkube/ExampleResource.java) 是一个标准的 Spring @ `[RestController](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html)`。它有一个响应**你好的请求映射。！！**对任何`GET`的请求。

## 将 web 应用程序部署到 Kubernetes 中

我们将从将示例 web 应用程序部署到 Kubernetes 开始；然后，我将向您展示如何进行一些调整并将其部署到 OpenShift 中。

### 步骤 1:构建应用程序

第一步是像构建任何其他 Maven web 应用程序项目一样构建项目。运行`mvn clean package`会在`target/example-0.0.0-SNAPSHOT.war`的目标目录下生成一个新的`war`工件。

出于这个例子的目的，我们将使用[迷你库贝](https://kubernetes.io/docs/setup/learning-environment/minikube/)。因此，我们可以从集群中提取映像，而不必将其推送到共享注册表中，我们将使用 Minikube 的`docker`守护进程`eval $(minikube docker-env)`。

我们现在可以发出`mvn k8s:build`命令来为我们的应用程序构建 docker 映像，如图 1 所示。

[![mvn k8s:build](img/02be3749555417e027a1d47db213ee66.png "Eclipse JKube k8s:build")](/sites/default/files/blog/2020/06/k8s-build.png)

Figure 1: Build the docker image for the application.

在我们的 docker 注册表中，新的 docker 图像将被标记为`webapp/example:latest`。

### 步骤 2:创建集群配置

接下来，我们创建所需的集群配置资源清单，并将它们应用到配置了`kubectl`的集群，如图 2 所示。

[![mvn k8s:resource k8s:apply](img/d3f664f59c5aa2a244f7c3ab2869b1ae.png "Eclipse JKube k8s:resource k8s:apply")](/sites/default/files/blog/2020/06/k8s-resource-apply.png)

Figure 2: Create and deploy the cluster configuration.

前面的命令将在`target/classes/META-INF/jkube/kubernetes.yml`中生成 Kubernetes 配置清单，并将它们应用到集群。

### 步骤 3:验证应用程序正在运行

最后，我们通过输入图 3 所示的命令来验证一切都在运行。

[![kubectl get pod](img/fd43b2c758f1c7d68abecb6d2bac4b1d.png "Eclipse JKube verify deployment")](/sites/default/files/blog/2020/06/verify-deployments.png)

Figure 3: Verify that the application is running.

## 将 web 应用程序部署到 OpenShift 中

按照与上一节相似的步骤，我们可以无缝地将示例 web 应用程序部署到 OpenShift 集群，只需使用`openshift-maven-plugin`而不是`kubernetes-maven-plugin`:

```
<build>
  <plugins>
    <plugin>
      <groupId>org.eclipse.jkube</groupId>
      <artifactId>openshift-maven-plugin</artifactId>
      <version>${version.jkube}</version>
    </plugin>
  </plugins>
</build>

```

在这种情况下，插件前缀是`oc`而不是`k8s`，但是要运行的目标是相同的。注意，在这种情况下，构建步骤使用 S2I 而不是 docker 来执行构建。

[![mvn oc:build](img/0709cfe1177a3b3bff22e6d82f463663.png "Eclipse JKube oc:build")](/sites/default/files/blog/2020/06/oc-build.png)

Figure 4: Build the container image using S2I binary build strategy for the application.

[![mvn k8s:resource k8s:apply](img/4d4942c09f98b38a4a06281b9b39f8bf.png "Eclipse JKube oc:resource oc:apply")](/sites/default/files/blog/2020/06/oc-resource-apply-verify.png)

Figure 5: Create and deploy the cluster OpenShift specific configuration.

## 开发者体验

您已经看到了 Eclipse JKube 如何让我们轻松地将 web 应用程序部署到云中。它还有其他的特性来减轻我们作为开发人员的负担。让我们看看 Eclipse JKube 如何增强典型的日志检索。

### 日志检索

如果我们想要打印刚刚部署的 web 应用程序的日志，我们可以简单地运行`mvn k8s:log`。这些日志将在当前控制台中被打印和跟踪(或跟踪),如图 6 所示。

[![mvn k8s:log](img/15f24ac5804465c68f7d52ebf9e621c0.png "Eclipse JKube k8s:log")](/sites/default/files/blog/2020/06/k8s-log.png)

Figure 6: Logs for the Kubernetes application deployment.

图 6 中的屏幕截图显示了我们刚刚在 Kubernetes 上部署的应用程序的日志。您可以看到 Apache Tomcat web 应用程序容器已经启动，并且 web 应用程序已经部署到根上下文中。

**注意**:默认情况下，Eclipse JKube 使用 [Apache Tomcat](https://tomcat.apache.org/) 作为其 web 应用程序容器，通过[JKube/JKube-Tomcat 9-binary-s2i](https://quay.io/repository/jkube/jkube-tomcat9-binary-s2i)基础映像。在后续文章中，我将向您展示如何使用不同的 web 应用程序容器(如 [Jetty](https://www.eclipse.org/jetty/) )，只需添加特定于容器的文件。

## 结论

在本文中，您看到了将一个老式的 Java web 应用程序转换成一个成熟的云原生应用程序是多么容易，只需将 Eclipse JKube 插件依赖项添加到您的 Maven POM 中。示例的完整源代码可在 [GitHub](https://github.com/marcnuri-demo/eclipse-jkube-webapp/tree/v0.0.0-tomcat) 上获得。

如果你有兴趣了解更多关于 Eclipse JKube 的知识，你可以访问我们的主 [GitHub 库](https://github.com/eclipse/jkube)、[网站](https://www.eclipse.org/jkube/)或 [Gitter 频道](https://gitter.im/eclipse/jkube)。别忘了在推特上关注我们！

*Last updated: September 27, 2022*