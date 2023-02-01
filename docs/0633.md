# 使用 Jcliff 管理 JBoss EAP/Wildfly

> 原文：<https://developers.redhat.com/blog/2019/11/06/managing-jboss-eap-wildfly-using-jcliff>

系统管理可能是一项艰巨的任务。不仅需要确定最终状态应该是什么，更重要的是，如何确保系统达到并保持这种状态。以自动化的方式这样做同样重要，因为可能有大量的目标实例。关于企业 [Java](https://developers.redhat.com/topics/enterprise-java/) 中间件应用服务器，这些实例通常使用一组基于 XML 的文件来配置。尽管可以手动配置这些文件，但大多数应用服务器都有一个或一组基于命令行的工具，使最终用户不必担心底层配置。WebSphere Liberty 包含了各种工具来管理这些资源，而 [JBoss](https://developers.redhat.com/products/eap/overview) 包含了 [jboss-cli](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.2/html/management_cli_guide/index) 工具。

尽管每个工具都实现了它的实用用例，因为它允许适当的服务器管理，但是它没有遵守自动化和配置管理的一个原则:幂等性。确保期望的状态并不等同于每次迭代都执行相同的动作。必须引入额外的情报。除了幂等性，配置管理的另一个核心原则是以声明方式表达值，并存储在版本控制系统中。

Jcliff 是一个基于 Java 的实用程序，它构建在 JBoss 命令行界面之上，允许以声明的方式表达服务器配置的预期目的，而这又可以存储在版本控制系统中。我们将提供 Jcliff 实用程序的概述，包括固有的好处、安装选项和展示使用的几个示例。

## 问题空间

在开始使用 Jcliff 之前，必须了解 JBoss 的底层架构。JBoss 服务器的配置可以用动态模型表示(DMR)符号来表示。以下是 DMR 的一个例子:

```
{
   "system-property"   =>   {
      "foo"      => "bar",
      "bah"      => "gah"
   }
}

```

上面的 DMR 例子描述了将存储在 JBoss 配置中的几个 Java 系统属性。通过执行以下命令，可以使用 JBoss CLI 添加这些属性:

```
/system-property=foo:add(value=bar)
/system-property=bah:add(value=gah)

```

挑战在于相同的命令不能执行多次。否则，将产生类似下面的错误。

```
{
    "outcome" => "failed",
    "failure-description" => "WFLYCTL0212: Duplicate resource [(\"system-property\" => \"foo\")]",
    "rolled-back" => true
}

```

Jcliff 的过人之处在于它包含了必要的智能来确定 JBoss 配置的当前状态，然后应用必要的适当配置。这对于在使用 JBoss 时采用适当的配置管理至关重要。

## 安装 Jcliff

对 Jcliff 有了基本的了解后，让我们讨论一下获得该实用程序的方法。首先，Jcliff 可以从[项目库](https://github.com/bserdar/jcliff)的源代码构建，因为它是一个免费的开源项目。或者，可以使用流行的软件包工具在每个主要的操作系统家族中安装 Jcliff。

### Linux(每分钟转数)

当能够使用包管理器使用 rpm 时，可以在 Linux 包上使用 Jcliff。

首先，安装 yum 存储库:

```
$ cat /etc/yum.repos.d/jcliff.repo
[jcliff]
baseurl = http://people.redhat.com/~rpelisse/jcliff.yum/
gpgcheck = 0
name = JCliff repository

```

一旦添加了这个存储库，就可以使用 [Yum](http://yum.baseurl.org/) 或 [Dnf](https://fedoraproject.org/wiki/DNF?rd=Dnf) 来安装 JCliff:

使用 yum:

```
$ sudo yum install jcliff

```

或者使用 dnf:

```
$ sudo dnf install jcliff

```

### 手动安装

Jcliff 也可以手动安装，而不需要利用包管理器。这对于那些不能使用 rpm 包的 Linux 发行版来说很有用。只需从项目存储库中下载并解压来自[发布工件](https://github.com/bserdar/jcliff/releases/download/v2.12.1/jcliff-2.12.1-dist.tar.gz)的归档文件。

```
$ curl -O \
https://github.com/bserdar/jcliff/releases/download/v2.12.1/jcliff-2.12.1-dist.tar.gz
$ tar xzvf jcliff-2.12.1-dist.tar.gz

```

将生成的 Jcliff 文件夹放在您选择的目标位置。这个位置被称为 JCLIFF_HOME。添加此环境变量，并将其添加到路径中，如下所述:

```
$ export JCLIFF_HOME=/path/to/jcliff-2.12.1/
$ export PATH=${JCLIFF_HOME}/bin:${PATH}

```

此时，您应该能够执行 jcliff 命令了。

### OSX

虽然 OSX 上的用户可以利用手动安装选项，但那些使用 brew 软件包管理器的用户也可以使用该工具。

执行以下命令，使用 Brew 安装 Jcliff:

```
$ brew tap redhat-cop/redhat-cop
$ brew install redhat-cop/redhat-cop/jcliff

```

### Windows 操作系统

不要害怕，Windows 用户；您也可以利用 Jcliff，而不必从源代码编译。和 OSX 一样，Windows 也没有官方的软件包管理器，但是巧克力公司被赋予了这个角色。

执行以下命令，使用 Chocolatey 安装 Jcliff:

```
$ choco install jcliff

```

## 使用 Jcliff

在您的机器上正确安装了 Jcliff 之后，让我们通过一个简单的例子来演示该工具的使用。如前所述，Jcliff 利用描述目标配置的文件。此时，您可能会有这样的问题:这些配置的格式是什么，从哪里开始？

让我们举一个简单的例子，看看如何向 JBoss 配置添加一个新的系统属性。启动 JBoss 实例，并使用 JBoss 命令行界面进行连接:

```
$ /bin/jboss-cli.sh --connect
[standalone@localhost:9990 /] ls /system-property
[standalone@localhost:9990 /] 

```

现在，使用 Jcliff 来更新配置。首先，我们需要创建一个 Jcliff 规则。该规则类似于 DMR 格式，如下所示:

```
$ cat jcliff-prop
{ "system-property" => {
  "jcliff.enabled" => "true",
  }
}

```

然后我们可以简单地要求 JCliff 对我们的 JBoss 服务器运行这个脚本:

```
$ jcliff jcliff-prop
Jcliff version 2.12.4
2019-10-02 17:03:40:0008: /core-service=platform-mbean/type=runtime:read-attribute(name=system-properties)
2019-10-02 17:03:41:0974: /system-property=jcliff.enabled:add(value="true")

```

使用 JBoss CLI 验证更改是否已应用。

```
$ “${JBOSS_HOME}/bin/jboss-cli.sh” --connect --command=/system-property=jcliff.enabled:read-resource
{
	"outcome" => "success",
	"result" => {"value" => "true"}
}

```

要演示 Jcliff 如何处理重复执行，请再次运行前面的 Jcliff 命令。检查输出。

```
$ jcliff jcliff-prop
Jcliff version 2.12.4
2019-10-02 17:05:34:0107: /core-service=platform-mbean/type=runtime:read-attribute(name=system-properties)

```

注意，只对 JBoss 服务器执行了一个命令，而不是两个。该措施是评估 JBoss 内部当前状态的结果，并确定无需采取任何措施即可达到所需状态。任务完成。

## 后续步骤

虽然前面的场景并不太复杂，但是您现在应该已经掌握了理解 Jcliff 实用程序的能力和功能以及通过声明性配置和自动化提供的好处所必需的知识。然而，当构建企业级系统时，Jcliff 不会被手动执行。您可能希望将该实用程序集成到一个适当的配置管理工具中，该工具采用了前面描述的许多自动化和配置原则。幸运的是，Jcliff 已经被集成到几个流行的配置管理工具中。

在下一篇文章中，我们将提供 Jcliff 可用的配置管理选项的概述，以及允许您快速建立对 Jcliff 的企业级信心的示例。

*Last updated: July 1, 2020*