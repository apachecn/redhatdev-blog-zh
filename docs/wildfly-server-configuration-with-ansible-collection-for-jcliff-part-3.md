# 带有 JCliff 的 Ansible 集合的 WildFly 服务器配置，第 3 部分

> 原文：<https://developers.redhat.com/blog/2020/12/21/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-3>

欢迎阅读这个由三部分组成的系列文章的最后一部分，这一部分将介绍如何使用 JCliff 的 Ansible Collection 来管理 WildFly 或[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/download)(JBoss EAP)实例。之前，我们已经讨论过[使用其基本特性](https://developers.redhat.com/blog/2020/11/06/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-1/)安装和配置 JCliff Ansible 集合和[。在本文中，我们将讨论项目的最新版本](https://developers.redhat.com/blog/2020/12/03/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-2/)中可用的高级选项。事不宜迟，我们开始吧！

## 使用自定义 JCliff 规则

JCliff 支持调整和配置大量的 WildFly 子系统。其中许多尚未集成到 Ansible 集合中。然而，仍然有可能利用这些额外的功能。在这一节中，我们将向您展示如何实现一个定制的规则集来利用 Ansible 中 JCliff 的全部功能。

### 配置 WildFly 的邮件子系统

假设我们想在 WildFly 配置中配置一个邮件会话。目前，`jcliff:`模块没有可用的邮件元素。相反，我们可以使用 JCliff 项目提供的脚本:

```
{
  "mail" => {
     "mail-session" => {
        "testSession" => {
           "from" => "a@b.com",
           "jndi-name" => "java:jboss/mail/testSession",
           "server" => {
              "smtp" => {
                 "outbound-socket-binding-ref" => "mail-smtp",
                 "ssl" => false
               }
           }
         }
     }
  }
}
```

我们可以将这个片段提供给 Ansible collection，让 JCliff 执行，从而配置我们的 WildFly 服务器的邮件子系统。首先，让我们为 JCliff 配置文件创建一个目录，并下载示例:

```
vars:
...
  jcliff_config_files_dir: /opt/jcliff
...
     - name: Create directory for jcliff rules
       file:
         name: "{{ jcliff_config_files_dir }}"
         state: directory

     - name: Download mail.test configuration file for JCLiff
       uri:
         url: https://raw.githubusercontent.com/bserdar/jcliff/master/testscripts/mail.test
         dest: "{{ jcliff_config_files_dir }}/mail.test"
         creates: "{{ jcliff_config_files_dir }}/mail.test"

```

接下来，我们配置`jcliff:`模块来执行这个规则集目录中的可用文件:

```
...
- jcliff:
   wfly_home: "{{ jboss_home }}"
   rule_file: "{{ jcliff_config_files_dir }}"
   subsystems:
...
```

**注意**:JCliff 的 Ansible 集合的下一个版本将包含一个 mail 元素。

### 验证新配置

很容易验证配置更改是否按预期发生。WildFly 的日志文件应该包含一条消息，提到新绑定的邮件会话:

```
...
14:38:36,372 INFO  [org.jboss.as.mail.extension] (MSC service thread 1-6) WFLYMAIL0001: Bound mail session [java:jboss/mail/testSession]
...
```

有关邮件会话配置的更多信息，我们可以使用 WildFly 的 JBoss CLI 输入一个简单的查询:

```
/subsystem=mail/mail-session=testSession:read-resource
{

    "outcome" => "success",
    "result" => {
        "debug" => false,
        "from" => "a@b.com",
        "jndi-name" => "java:jboss/mail/testSession",
        "custom" => undefined,
        "server" => {"smtp" => undefined}
    },
    "response-headers" => {"process-state" => "reload-required"}
}

```

请注意，这最后一个更改需要重新加载 WildFly 的配置。如果你已经在系统上部署了 WildFly 作为服务，你也可以要求 Ansible 重启它。如果 WildFly 没有作为服务部署，您仍然可以使用 Ansible 中的 JBoss CLI 来重新加载:

```
- name: "Restart WildFly"
  command: "{{ jboss_home }}/bin/jboss-cli.sh --connect --command=':reload'"

```

## JCliff 故障排除

当然，并不是所有东西都像预期的那样开箱即用。有时，您可能需要分析和调查幕后真正发生的事情。在这一节中，我们将重点介绍如何在使用 JCliff 和 Ansible 时解决问题。

首先，请记住 Ansible 只生成 JCliff 所需的配置文件，并运行命令行工具。因此，在对 JCliff 使用 Ansible 集合时，几乎不会出错。最坏的情况是，您可能会遇到无效路径或类似的人为错误。使用带有详细日志记录的 Ansible(比如`-vvvv`)应该可以帮助您识别这样的问题。

如果遇到特定于`jcliff:`模块的挑战，通常最好在 Ansible 之外运行该工具，看看发生了什么。在这种情况下，您的第一步是检索 Ansible 生成的配置文件，并确保内容是正确的。

每次 Ansible 运行后，都会删除为 JCliff 生成的配置文件。要访问它们，您必须指示 Ansible 不要删除这些文件。在运行 Ansible 之前设置以下参数:

```
export ANSIBLE_KEEP_REMOTE_FILES=1

```

当`jcliff:`模块出现故障时，您应该能够根据故障子系统的名称找到配置文件。例如，如果您对 JDBC 驱动程序部署的自动化有问题，您应该寻找一个名为`drivers-0.jcliff.yml`的文件。

找到文件后，运行带有选项`-v`的`jcliff`，并向其传递以下配置文件:

```
$ jcliff -v ~/.ansible/tmp/ansible-tmp-1597756555.46-5332-269490407528192//drivers-0.jcliff.yml

```

如果在 JCliff 执行过程中没有发现问题，那么所执行的 JBoss CLI 查询集应该仍然可以让您了解发生了什么。然后，您可以连接到 WildFly 实例(使用服务器提供的 JBoss CLI 脚本)并直接运行这些查询。这样做很可能会发现潜在的问题。

## 结论

在发布 JCliff 的 Ansible collection 之前，需要做大量的准备工作来自动化 WildFly 实例的设置和配置。即使您直接使用 JCliff，您仍然需要一些配置文件。如果没有 JCliff，您需要设计、编写和测试数百行 JBoss CLI 脚本。一些开发人员试图通过使用自动化模板来解决这个限制。不幸的是，如果在 Ansible 之外修改配置文件，该变通办法会导致问题。

JCliff 的 Ansible 集合解决了这些问题。它很简洁，并且可以平滑地集成到 Ansible 中。作为一名开发人员，您可以用几行代码陈述所需的配置，就像您对 Ansible 管理的任何其他资源所做的那样。

我们希望您喜欢这个系列来了解 JCliff 的 Ansible collection，并希望您下次需要配置 WildFly 或 JBoss EAP 实例时尝试一下这个工具。

## 承认

特别感谢 Andrew Block 回顾了这一系列的文章。

*Last updated: December 20, 2020*