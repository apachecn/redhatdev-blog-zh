# 用 Kogito 创建您的第一个应用程序

> 原文：<https://developers.redhat.com/blog/2019/08/29/create-your-first-application-with-kogito>

[Kogito](https://kogito.kie.org/) 是用于构建智能应用的云原生开发、部署和执行平台，以久经考验的功能为后盾。它源于众所周知的开源项目，如 Drools 和 jBPM。Kogito 的主要特征包括:

*   **云优先—** 云部署是首要的目标运行时环境。
*   **特定领域—** 不再向您的客户端应用程序泄露服务背后的技术抽象。
*   **增强开发人员的能力** **—** Kogito 基于久经考验的组件提供强大的开发人员体验。

## 创建您的第一个应用程序

本文涵盖:

*   用 modeler 插件配置您的 IDE。
*   生成新项目。
*   构建和运行项目。

### **安装带建模插件的 Eclipse】**

为了能够利用流程的可视化建模，请从 Marketplace 下载 Eclipse IDE 并安装

*   Eclipse BPMN2 Modeler 插件(带有 jBPM 运行时扩展)

**注意:**目前，Eclipse IDE 是唯一完全支持流程和规则建模的 IDE，但是该团队正在努力将基于 web 的可嵌入 BPMN2、DMN 和场景编辑器引入 VSCode 和 Eclipse Che。

要构建自己的由 Kogito 支持的服务，最好的方法是从使用 Maven 原型生成项目开始。

## SpringBoot

### 生成项目

```
mvn archetype:generate -DarchetypeGroupId=org.kie.kogito -DarchetypeArtifactId=kogito-springboot-archetype -DarchetypeVersion=0.1.3 -DgroupId=com.company -DartifactId=sample-kogito
```

项目中包含了现成的工具，允许测试生成的应用程序。只需用 Maven 构建项目并启动应用程序。

和以前一样，您可以选择使用 SpringBoot Maven 插件或作为普通的 Java 服务来启动它。在这个例子中，我们将使用 SpringBoot Maven 插件。

### **启动您的应用程序**

打开命令行，进入新创建的 sample-kogito 文件夹。

```
mvn clean package spring-boot:run
```

### 实现您的应用程序逻辑

现在您已经准备好实现您的业务逻辑，它可以由业务流程、规则、决策表、Java 服务等等组成。

### 在 JVM 模式下构建和运行

```
mvn clean package
java -jar target/jbpm-springboot-example-{version}.jaror on Windows
java -jar target\jbpm-springboot-example-{version}.jar
```

#### 了解更多信息

[开发现场:介绍 Kogito](https://developers.redhat.com/blog/2019/07/23/devnation-live-introducing-kogito/)

*Last updated: September 3, 2019*