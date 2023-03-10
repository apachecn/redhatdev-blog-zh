# mandrel:quar kus 专用的 GraalVM 发行版

> 原文：<https://developers.redhat.com/blog/2021/04/14/mandrel-a-specialized-distribution-of-graalvm-for-quarkus>

当我们第一次宣布[心轴](https://github.com/graalvm/mandrel)的时候，我们解释了为什么红帽需要一个[GraalVM](/blog/2020/06/05/mandrel-a-community-distribution-of-graalvm-for-the-red-hat-build-of-quarkus/)的下游分布。我们对 GraalVM 的[原生映像能力](https://www.graalvm.org/reference-manual/native-image/)最感兴趣，特别是在 [Quarkus](/products/quarkus/getting-started) 的上下文中。在本文中，我们解释什么是心轴，什么不是心轴。我们将介绍 Mandrel 的一些技术特性，并提供一个使用 Mandrel 和 Quarkus 的简短演示。

## 什么是心轴？

Mandrel 专注于 GraalVM 的`native-image`组件，以便为 Quarkus 用户提供一种为他们的应用程序生成原生映像的简单方法。使用 Quarkus 的开发者应该能够从 [Java](/topics/enterprise-java) 源代码一直到运行在 [Linux](/topics/linux) 上的精简的、本地的、依赖于平台的应用程序。这种能力对于在云原生应用开发模型中部署到容器至关重要。

我们有意不承诺 GraalVM 在 Mandrel 中的其他用途。GraalVM 的完整发行版[比原生映像](https://www.graalvm.org/docs/graalvm-as-a-platform/)多得多:它有[多语言支持](https://www.graalvm.org/reference-manual/polyglot-programming/)；Truffle 框架，支持高效的解释器实现；一个用于`native-image`的 [LLVM 编译器后端](https://www.graalvm.org/reference-manual/native-image/LLVMBackend/)；libgraal JIT 编译器作为 HotSpot 的 C2 服务器编译器的替代品；还有更多。Mandrel 是 GraalVM 功能的一小部分。我们支持 Quarkus 等项目中的`native-image`用例。

## 为什么是心轴？

Mandrel 让我们去掉 GraalVM 中不需要的部分，专注于`native-image`。它减少了我们必须维护和测试的代码库。Mandrel 让我们使用上游 OpenJDK 11，而不是 [Labs OpenJDK 11](https://github.com/graalvm/labs-openjdk-11) ，这是用于 [GraalVM CE](https://github.com/graalvm/graalvm-ce-builds/releases) 的基础 Java 开发套件。Red Hat 有一个团队支持面向 Red Hat 客户的 OpenJDK，因此使用上游 OpenJDK 是 Mandrel 的天然选择。Red Hat 也有参与 OpenJDK upstream 的悠久历史，目前领导着 [jdk-updates OpenJDK 项目](https://openjdk.java.net/projects/jdk-updates/)的 OpenJDK 11u 流。

通过我们在这些项目中的领导地位，我们做出了稳定的改变，改进了 GraalVM，从而改进了 Mandrel。因为 Red Hat 是一家上游优先的公司，我们试图从一开始就在上游获得补丁。到目前为止，我们已经为 Linux 和 Windows 的 [GraalVM CE 增加了调试功能](/blog/2020/06/25/debugging-graalvm-native-images-using-gdb/)(使用`-g`标志)，并且我们正在积极地与甲骨文实验室合作[将 JDK 飞行记录器支持](https://github.com/oracle/graal/pull/3070)添加到`native-image`。一旦集成，开发人员将能够使用 [JDK 飞行记录器](/blog/2020/08/25/get-started-with-jdk-flight-recorder-in-openjdk-8u/)来观察和跟踪他们的应用程序中的行为和性能挑战，无论它们是以 JVM 模式还是本机模式运行。

## 心轴中的 GraalVM 组件

为了通过 Mandrel 交付 GraalVM 的`native-image`特性，我们只构建了[底层 VM 框架](https://github.com/oracle/graal/blob/master/substratevm/README.md)，以及它的依赖项。这包括 [GraalVM SDK](https://github.com/oracle/graal/blob/master/sdk/README.md) 和 [GraalVM 编译器](https://github.com/oracle/graal/blob/master/compiler/README.md)。所有其他 GraalVM 组件，如[松露](https://github.com/oracle/graal/blob/master/truffle/README.md)和[苏龙](https://github.com/oracle/graal/blob/master/sulong/README.md)，都不是 Mandrel 的一部分，我们不构建它们。

除了 GraalVM SDK 和编译器之外，底层 VM 还依赖于基本 JDK 和本机编译器工具链。Mandrel 使用上游 OpenJDK 11 构建和 GNU 编译器集合(GCC)来满足这些依赖性，而 GraalVM 使用 [Labs OpenJDK 11](https://github.com/graalvm/labs-openjdk-11) 并支持 GCC 和 LLVM 工具链。

我们倾向于将 Mandrel 视为 OpenJDK 的扩展——大部分是 JAR 文件，很少是本机二进制文件。事实上，我们的[产品化心轴图像](https://catalog.redhat.com/software/containers/quarkus/mandrel-20-rhel8/5f43d40b73bfe598d1857c15)只在 RHEL 基地的常规`java-11-openjdk-static-libs`安装基础上增加了两个 rpm。

## 建筑心轴

GraalVM 是使用用 Python 编写的工具 [mx](https://github.com/graalvm/mx) 构建的。不同 GraalVM 组件之间的所有依赖关系都是通过 *mx 套件*定义的，部分构建过程被定义为 *mx 扩展*。因此，`mx`也是心轴构建过程中不可或缺的一部分。扩展`mx`套件以支持 GraalVM 和心轴构建[被证明是困难的](https://github.com/oracle/graal/issues/2502)。将这项工作与减少正在构建的组件和拆分构建步骤结合起来尤其具有挑战性，我们这样做是为了支持在不同的构建系统上构建不同的部分。

我们目前的解决方案是使用[芯棒包装](https://github.com/graalvm/mandrel-packaging)来制造芯棒。这个工具处理从其他需要的`mx`套件中移除不必要的依赖，并且它以正确的顺序调用必要的命令。`mandrel-packaging`工具还让我们分三步构建 Mandrel:首先，我们生成 Java 部件(JAR 文件和 Maven 工件)；然后，原生部分(GraalVM 的静态库)；最后，我们用 OpenJDK 11 将它们组装起来，产生一个支持`native-image`的运行时。

以下是使用 OpenJDK 11 作为基础，从源代码构建 Mandrel 当前版本的基本说明:

```
$ curl -sL https://github.com/AdoptOpenJDK/openjdk11-upstream-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-jdk_x64_linux_11.0.10_9.tar.gz -o jdk.tar.gz
$ curl -sL https://github.com/AdoptOpenJDK/openjdk11-upstream-binaries/releases/download/jdk-11.0.10%2B9/OpenJDK11U-static-libs_x64_linux_11.0.10_9.tar.gz -o jdk-static-libs.tar.gz
$ export JAVA_HOME=$(pwd)/openjdk-11
$ mkdir ${JAVA_HOME}
$ tar xf jdk.tar.gz -C ${JAVA_HOME} --strip-components=1
$ tar xf jdk-static-libs.tar.gz -C ${JAVA_HOME} --strip-components=1
$ git clone --depth 1 --branch 5.279.1 https://github.com/graalvm/mx
$ git clone --depth 1 --branch mandrel-21.0.0.0-Final \
  https://github.com/graalvm/mandrel 
$ git clone --depth 1 --branch mandrel-21.0.0.0-Final \
  https://github.com/graalvm/mandrel-packaging
$ $JAVA_HOME/bin/java -ea ./mandrel-packaging/build.java \
  --mx-home $(pwd)/mx --mandrel-repo $(pwd)/mandrel
$ ./mandrel-java11-21.0.0.0-Final/bin/native-image --version
GraalVM Version 21.0.0.0-Final (Mandrel Distribution) (Java Version 11.0.10+9)

```

## 与 Oracle 实验室的合作

GraalVM CE 11 基于 [Labs OpenJDK 11](https://github.com/graalvm/labs-openjdk-11/) ，是上游 OpenJDK 11 的一个分支，用于构建支持 libgraal 和 GraalVM CE 的基础 JDK。Labs OpenJDK 11 由构建 [GraalVM CE](https://github.com/graalvm/graalvm-ce-builds/releases) 的同一个团队维护。

因为 Graal VM CE 基于 Labs OpenJDK 11，Graal VM 团队可以利用 Java 11 中最新的 [Java 级 JVM 编译器接口](https://openjdk.java.net/jeps/243) (JVMCI)增强和*增强*它们。因为这两个项目联系如此紧密，当我们开始的时候，[不可能用上游的 OpenJDK 11](https://github.com/oracle/graal/issues/2196) 代替 Labs OpenJDK 11 作为构建 JDK。我们与 Oracle 实验室合作实现了心轴。

我们希望用普通的 OpenJDK 11 构建，而 Oracle 实验室希望下游维护的补丁更少。这项工作以一组补丁合并到 GraalVM 代码库和上游 OpenJDK 11 中而结束。这是进一步合作的良好基础。最值得注意的是，我们需要支持将 OpenJDK 库作为 OpenJDK 11 的静态变体来构建。Mandrel 使用 OpenJDK 静态库，而不是常规 OpenJDK 发行版中的共享库。这让我们可以将本地 Java 应用程序链接到依赖于 OpenJDK 库的本地映像中。

## 使用带 quartus 的心轴

在撰写本文时，Quarkus 是 Mandrel 的主要消费者。下面是使用 Mandrel 为用 Java 编写的简单 RESTEasy Quarkus 应用程序生成本机可执行文件的快速演示:

```
$ echo | mvn io.quarkus:quarkus-maven-plugin:1.12.2.Final:create
$ cd code-with-quarkus
$ ./mvnw -Dnative verify \
         -Dquarkus.native.container-build=true \
         -Dquarkus.native.builder-image=quay.io/quarkus/ubi-quarkus-mandrel:20.3-java11
$ ./target/code-with-quarkus-1.0.0-SNAPSHOT-runner  
__  ____  __  _____   ___  __ ____  ______ 
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/  
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \    
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/    
2021-03-08 17:57:40,084 INFO  [io.quarkus] (main) code-with-quarkus 1.0.0-SNAPSHOT native (powered by Quarkus 1.12.2.Final) started in 0.014s. Listening on: http://0.0.0.0:8080
2021-03-08 17:57:40,084 INFO  [io.quarkus] (main) Profile prod activated.  
2021-03-08 17:57:40,084 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
$ curl -w "\n" http://localhost:8080/hello-resteasy
Hello RESTEasy

```

我们首先在目录中创建一个简单的 RESTEasy Quarkus 应用程序，`code-with-quarkus`。然后，我们用 Maven 包装器(`mvnw`)编译 Java 源代码，并使用预定义的容器映像从 Java 类文件构建本机可执行文件。`-Dnative`参数指示 Maven 生成本地可执行文件，`-Dquarkus.native.container-build=true`指示 Quarkus 使用构建器映像构建本地可执行文件，`-Dquarkus.native.builder-image=...`指示[使用哪个构建器映像](https://quay.io/repository/quarkus/ubi-quarkus-mandrel?tag=latest&tab=tags)。请访问 [Quarkus 指南](https://quarkus.io/guides/building-native-image)了解关于此示例的更多详细信息。

生成的二进制文件`./target/code-with-quarkus-1.0.0-SNAPSHOT-runner`，是一个本地 ELF 二进制文件，适合在基本的 Linux 发行版上运行，就像一个[最小 UBI 8 镜像](https://catalog.redhat.com/software/containers/ubi8/ubi-minimal/5c359a62bed8bd75a2c3fba8):

```
$ file target/code-with-quarkus-1.0.0-SNAPSHOT-runner
target/code-with-quarkus-1.0.0-SNAPSHOT-runner: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=b43068dce7c114a0e8d018370a9d47e1e257de44, not stripped

```

图 1 显示了用 Mandrel 的`native-image`可执行文件封装 Quarkus 应用程序的工作流程。

[![A diagram of the workflow to produce a Quarkus container image.](img/575a74c698ec8e0555715bc951442434.png "containerize")](/sites/default/files/blog/2021/04/containerize.jpeg)

Figure 1: Containerizing a Quarkus application with Mandrel.

## 摘要

Mandrel 是 GraalVM 的下游版本，专注于 Red Hat 支持的项目中 Java 应用程序的原生映像编译。因此，它不是所有用户的 GraalVM `native-image`替代品。例如，Mandrel 不附带 polyglot 或 LLVM 工具链支持，而一些用例可能需要这种支持。然而，Red Hat 工程师为上游 GraalVM 做出了贡献，并努力保持上游 GraalVM 代码的最新状态。

Mandrel 是 GraalVM GitHub 组织的一部分，并作为该组织的一部分进行维护；这是 GraalVM 生态系统的一部分，我们不打算改变它。也就是说，Mandrel 旨在满足 Quarkus 等项目的可持续性需求。我们在这方面的主要目标是保持作为更大的 Quarkus 生态系统一部分的心轴版本的稳定，并在该生态系统的整个生命周期中得到支持。因此，我们很可能会将某些心轴版本的更新保持得比常规的上游 GraalVM CE 版本更长。此外，因为 Mandrel 依赖于 OpenJDK 11，我们需要用 OpenJDK 的季度安全补丁来保持它的最新。

请定期查看 GitHub 页面上的[心轴发布](https://github.com/graalvm/mandrel/releases/)。我们目前有 Linux 和 Windows 的下载。我们通常在 GitHub 发布的同时或之后不久，向 [quay.io](https://quay.io/repository/quarkus/ubi-quarkus-mandrel) 发布基于 UBI 8 的 Linux 容器映像。

*Last updated: October 14, 2022*