# 在 RHEL 7 和 OpenJDK 8 上使用 Java 构建您的第一个应用

> 原文：<https://developers.redhat.com/articles/using-java-rhel-7-openjdk-8>

在不到 10 分钟的时间内开始在 Red Hat Enterprise Linux 上使用 Java 8 进行开发。

### 简介和先决条件

在本教程中，您将看到如何通过创建一个简单的 Hello World 应用程序开始在 Red Hat Enterprise Linux 上进行 Java 开发。您将安装一个 Java 开发工具包(JDK)，并了解哪些 Java 包是可用的。完成整个教程不到 10 分钟。

您将需要一个 Red Hat Enterprise Linux 7 系统，该系统具有当前的 Red Hat 订阅，允许您从 Red Hat 下载软件和更新。如果您没有红帽企业版 Linux 订阅，请在 developers.redhat.com[注册](https://developers.redhat.com/)后获得[红帽企业版 Linux 开发者套件](https://developers.redhat.com/products/rhel/overview/)。

如果您在任一点遇到困难，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 1.准备好你的系统

*2 分钟*

在这一步中，您将学习如何使用`yum`包管理工具来找出哪些 Java 包可用，并为您的系统下载和安装更新。您还将看到如何启用对更多软件包的附加软件存储库的访问。

首先，从*应用*菜单启动一个*终端*窗口。然后使用`su -`切换到根用户 ID，并使用`subscription-manager`验证您可以访问 Red Hat 软件库。

```
$ su -
# subscription-manager repos --list-enabled
```

如果您没有看到任何已启用的存储库，您的系统可能没有注册到 Red Hat，或者可能没有有效的订阅。更多信息参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

### 下载并安装更新

现在通过运行`yum update`下载并安装任何可用的更新。如果有可用的更新，`yum`将列出它们并询问是否可以继续。

`# yum update`

### 确定是否已经安装了任何 Java 包

您的系统可能安装了一个或多个 Java 运行时环境(JRE)。可以看到带有`yum list installed`的已安装包列表。您可以缩小列表，只显示以 java 开头的包名。

`# yum list installed java\*`

输出包括软件包名称、版本信息和安装它的软件存储库。

如果安装了 JRE，可以跳过以下步骤。安装 JDK 时，也会安装 JRE。

要判断`java`是否在您的`PATH`中，请使用`which`:

`# which java`

如果`java`在您的路径中，请确定是哪个版本:

`# java -version`

### 查看可用的 JDK

您可以查看或搜索可与`yum list available`一起安装的软件包:

`# yum list available java\*devel`

输出将显示软件包名称及其所在的软件存储库。结果中仅包含已启用下载的软件存储库。

### 启用附加软件存储库

对于 Red Hat Enterprise Linux，有许多附加的软件存储库。这些在默认情况下是不启用的，因为它们中的包的支持策略不同于主要的 Red Hat Enterprise Linux 包。其中一些存储库包含 Java 开发人员感兴趣的包:

*   可选的 RPMs 库包括许多 Java 开发工具和库。大多数 Java 开发人员都希望启用这个存储库。

*   IBM 的 JRE 和 JDK 版本可以在*补充 rpm*库中找到。

*   Oracle JRE 和 JDK 位于第三方 Oracle Java rpm 存储库中。

下面提供了命令行的说明。如果您的系统安装了图形桌面，您可以使用图形版本的`subscription-manager`。红帽订阅管理器可以从*应用*菜单的*系统工具*组中启动。或者，您可以通过在命令提示符下键入`subscription-manager-gui`来启动它。从 subscription manager 的*系统*菜单中选择*存储库*。

注意:Red Hat 软件存储库的命名特定于您的系统所使用的 Red Hat Enterprise Linux 的服务器、工作站或桌面版。以下示例适用于服务器安装。如果您使用的是工作站或桌面版，在以下命令中用`-workstation-`或`-desktop-`代替`-server-`。

要查看当前启用了哪些存储库:

`# subscription-manager repos --list-enabled`

您还可以获得未启用的可用存储库的列表:

`# subscription-manager repos --list-disabled`

启用可选的 RPMs 存储库:

`# subscription-manager repos --enable rhel-7-server-optional-rpms`

启用一个存储库后，当您发出一个`yum`命令时，它将与其他已启用的存储库一起被搜索。

## 2.设置您的开发环境

*2 分钟*

在下一步中，您将成为一名 JDK。您应该仍然打开先前的*终端*窗口，并且仍然在`su`下运行。

首先，查看要安装的可用 JDK 列表:

`# yum install available java\*devel`

JDK 包的命名约定是`java-*version*-*provider*-devel_`。OpenJDK 的 1.8.0 版本命名为`java-1.8.0-openjdk-devel`。JRE 组件单独包装。JRE 包名与没有`-devel`的 JDK 相同。当您安装 JDK 软件包时，它会自动安装相应的 JRE。

使用以下命令安装 JDK，必要时更改版本号:

`# yum install java-1.8.0-openjdk-devel`

检查`javac`现在是否在您的路径中，并检查哪个版本:

`# javac -version`

### 管理 Java 版本

可以同时安装多个版本的 JRE 和 JDK 软件包。您的系统中可能安装了一组需要多个 Java 版本的应用程序或服务。红帽企业版 Linux 的 JRE 和 JDK 包安装在`/usr/lib/jvm`下的单独目录中。这使得它们可以同时安装。然而，在 shell 的命令路径中，一次只能有一个版本作为`java`或`javac`。

注意:按照最佳实践打包为 RPM 的 Java 应用程序或服务将指定必要的 JRE 版本作为包依赖项。这将导致`yum`找到并安装所需的特定 JRE。应用程序或服务将使用特定 JRE 的完整路径，而不是依赖于 shell 的命令搜索路径。

通过使用系统的`alternatives`命令或使用环境变量来改变路径，可以选择当您键入`java`或`javac`时使用哪个版本。`alternatives`命令将做出适用于整个系统的更改。对于一个开发系统来说，这是一个合理的选择。对于一个共享系统，比如运行多个应用程序的服务器，改变`alternatives`可能会产生不希望的副作用。

使用`alternatives`设置默认 JDK:

`# alternatives --config javac`

使用`alternatives`设置默认 JRE:

`# alternatives --config java`

请注意，许多相关的 Java 命令将同时更改。如需完整列表，请使用`alternatives --display`:

```
# alternatives --display javac
# alternatives --display java
```

使用环境变量，可以为当前会话、特定用户、系统范围或特定应用程序设置 PATH 和 JAVA_HOME。

### 设置`JAVA_HOME`

对于需要`JAVA_HOME`环境变量的工具，将其设置为`/usr/lib/jvm/*java-version*`。例如，要指定 OpenJDK 1.8.0 JDK，在您的脚本和/或构建配置中使用`JAVA_HOME=/usr/lib/java-1.8.0-openjdk`。在`/usr/lib/jvm`下有几种排列，包括 JRE 或 JDK 的全名，直到具体的补丁号，或者逐渐更通用的引用，如`java-1.8.0-openjdk`、‘Java-1 . 8 . 0’或仅仅是`java`。

如果您需要帮助，请参见[故障排除和常见问题解答](#TroubleshootingandFAQ3)。

## 3.Hello World 和您的第一个应用程序

*2 分钟*

在这一步中，您将使用命令行创建并编译一个简单的 Java 应用程序。如果没有打开*终端*窗口，从*应用*菜单启动。你应该在你的普通用户 ID 下运行，如果你仍然作为根用户运行，输入`exit`。

首先，您需要使用您喜欢的文本编辑器创建`Hello.java`，比如`vi`、`nano`或`gedit`:

`$ nano Hello.java`

将以下文本添加到文件中:

Hello.java

```
public class Hello {

    public static void main(String[] args) {
        System.out.println("Hello, Red Hat Developers World from Java " +
         System.getProperty("java.version"));
    }
}
```

现在用`javac`编译:

`$ javac Hello.java`

如果编译没有错误，运行它:

```
`$ java Hello
Hello, Red Hat Developers World from Java 1.8.0_71
```

### 接下来去哪里？

*   使用 [Ticket Monster](https://developers.redhat.com/ticket-monster/) 深入 Java 企业应用程序开发，这是一个中等复杂的应用程序，演示了如何使用 JBoss web 技术构建现代应用程序。

*   请访问 Red Hat Developers 网站，了解关于企业 Java 和 JBoss 技术的更多信息。

## 故障排除和常见问题

1.  作为一名开发者，如何获得红帽企业版 Linux 订阅？

    如果您没有红帽企业版 Linux 订阅，请在[developers.redhat.com](https://developers.redhat.com/)注册，然后下载[红帽企业版 Linux 开发者套件](https://developers.redhat.com/products/rhel/download)。

2.  哪里可以找到 Java 开发工具作为红帽企业 Linux 的`ant`、`maven`等库？

    找不到很多红帽企业 Linux 的 Java 包，应该去哪里找？

    许多 Java 开发工具和库都位于*可选 rpm*存储库中。默认情况下，*可选 RPMs* 存储库是不启用的，因为其中的包的支持策略不同于主要的 Red Hat Enterprise Linux 包。本教程的第 1 步展示了如何启用*可选 RPMs* 存储库，这是推荐给 Java 开发人员的。

    以下命令将启用存储库:

    `# subscription-manager repos --enable rhel-7-server-optional-rpms`

    现在您可以安装`ant`、`maven`和其他 Java 开发工具:

    `# yum install ant maven`

3.  Red Hat Enterprise Linux 有 Eclipse 这样的 Java 交互式开发环境(IDE)吗？

    JBoss Developer Studio 构建于 Eclipse 之上，它为您的整个开发生命周期提供了卓越的支持。它包含的特性将帮助您快速开始开发 Java 应用程序。出于开发目的，在[developers.redhat.com](https://developers.redhat.com/)注册后可获得 0 美元订阅

    更多信息参见 [JBoss Developer Studio 概述](https://developers.redhat.com/products/codeready-studio/overview)。

4.  我如何知道已经安装了哪些 JRE 和 JDK？

    可以看到带有`yum list installed`的已安装包列表。您可以缩小列表，只显示以 java 开头的包名。JDK 包的命名约定是`java-*version*-*provider*-devel_`。OpenJDK 的 1.8.0 版本被命名为`java-1.8.0-openjdk-devel`。JRE 组件单独包装。JRE 包名与不带`-devel`的 JDK 相同。

    `# yum list installed java\*`

    输出包括软件包名称、版本信息和安装它的软件存储库。

    如果您想为已经安装的 JRE 安装匹配的 JDK，请在包名中添加`-devel`。例如，如果您有`java-1.8.0-openjdk` JRE，您可以使用以下命令添加 JDK 组件:

    `# yum install java-1.8.0-openjdk-devel`

5.  为什么我的系统上已经安装了 OpenJDK JRE？

    一些 Red Hat 包需要一个 JRE。最常见的是互联网浏览器包，它安装了 Firefox 和 OpenJDK，以便能够运行 Java 小程序。

6.  OpenJDK 之外的 JRE/JDK 是否可用于红帽企业版 Linux？

    可以在红帽企业版 Linux 上安装甲骨文的 JRE/JDK 吗？

    OpenJDK、IBM 和 Oracle JRE/JDK 可从 Red Hat 软件仓库获得，以便通过`yum`轻松安装。IBM 和 Oracle 包位于可选的存储库中，默认情况下不启用。为 IBM 包启用*补充 RPMS* 存储库，或者为 Oracle 包启用*第三方 Oracle Java rpm*存储库。

    如果您安装了图形桌面，可以从*应用程序*菜单的*系统工具*组启动 Red Hat Subscription Manager。从 subscription manager 的*系统*菜单中选择*存储库*。

    要从命令行启用其他存储库，请在使用`su -`更改为 root 用户 ID 后运行以下一个或两个命令:

    ```
    # subscription-manager repos --enable rhel-7-server-supplementary-rpms
    ```

    或者

    ```
    # subscription-manager repos --enable rhel-7-server-thirdparty-oracle-java-rpms
    ```

    现在您可以使用`yum`查看可用 JDK 的列表:

    `yum list available java-\*devel`

    要安装 JDK，请使用:

    `yum install java-X.Y.Z-provider-devel`

    如果只安装 JRE，可以省略包名中的`-devel`。

    更多信息参见*[Oracle/Sun/IBM Java 包在哪里？](https://access.redhat.com/solutions/732883)上的*[红帽客户门户](https://access.redhat.com/)。

7.  我可以在 Red Hat Enterprise Linux 上使用 Java.com 的 Oracle Java 包吗？

    是的，除了 Red Hat Enterprise Linux 提供的包之外，您还可以使用来自 http://java.com/的 Oracle Java 包。在链接上:http://java.com/可以找到 Oracle Java 打包成 RPM 文件，兼容红帽 Enterprise Linux、`yum`和`rpm`，红帽包管理器。下载标有“Linux x64 RPM”的 Java 包。使用`yum`安装软件包:

    `# yum localinstall *path_to_downloaded_java_rpm*`

    注意:Oracle 的 Java RPM 包将安装在`/usr/java`下，而不是`/usr/lib/jvm`下。

8.  我可以同时安装多个 JRE/JDK 吗？

    可以同时安装多个版本的 JRE 和 JDK 软件包。红帽企业版 Linux 的 JRE 和 JDK 包安装在`/usr/lib/jvm`下的单独目录中。这使得它们可以同时安装。然而，在 shell 的命令路径中，一次只能有一个版本作为`java`或`javac`。参见上面的*管理 Java 版本*。

9.  `JAVA_HOME`应该设置成什么？

    对于需要`JAVA_HOME`环境变量的工具，将其设置为`/usr/lib/jvm/*java-version*`。例如，要指定 OpenJDK 1.8.0 JDK，在您的脚本和/或构建配置中使用`JAVA_HOME=/usr/lib/java-1.8.0-openjdk`。在`/usr/lib/jvm`下有几种排列，包括 JRE 或 JDK 的全名，直到具体的补丁号，或者逐渐更通用的引用，如`java-1.8.0-openjdk`、‘Java-1 . 8 . 0’或仅仅是`java`。建议使用像`java-1.8.0-openjdk`这样的`JAVA_HOME`值，因为它允许您指定哪个 JVM，而不需要绑定到特定的补丁级别。

10.  红帽企业版 Linux JDK 包也包括 JRE 吗？

    JRE 和 JDK 封装在独立但互补的 RPM 中，以避免冗余。当您使用`-devel`包安装 JDK 时，匹配的 JRE 包将在必要时自动安装。设置`JAVA_HOME`指向其中一个`java-`目录将会选择 JDK 和 JRE 组件。如果您只想要 JRE 组件，将`JAVA_HOME`设置为`jre-1.8.0-openjdk`。

11.  当我键入`java`或`javac`时，我如何改变使用的 JRE/JDK？

    使用默认的`PATH`设置，键入`java`或`javac`将使用来自`/usr/bin/`的命令。这些由`alternatives`命令管理，以便在不同的包之间切换。更多信息见上面的*管理 Java 版本*。请注意，使用`alternatives`命令将改变系统范围内为任何没有明确指定使用哪个 JVM 的命令或应用程序使用哪个 JRE/JDK。这可能会对共享系统或服务器产生意想不到的副作用。

12.  文本编辑器`nano`没有安装，怎么安装？

    你可以使用任何你喜欢的文本编辑器来代替`nano`，比如`vi`、`gedit`或者`emacs`。要安装一个包，比如`nano`，用`su -`改成 root 用户 ID，然后用`yum install *packagename*`。

    ```
    # yum install nano
    ```

*Last updated: January 9, 2023*