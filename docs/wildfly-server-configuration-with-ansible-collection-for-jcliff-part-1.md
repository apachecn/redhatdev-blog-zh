# 带有 JCliff 的 Ansible 集合的 WildFly 服务器配置，第 1 部分

> 原文：<https://developers.redhat.com/blog/2020/11/06/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-1>

这个由三部分组成的系列将指导您使用 Ansible 来微调 WildFly 或[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/download)(JBoss EAP)服务器配置。我们将使用 JCliff 的最新版本的 [Ansible 集合来扩展 Ansible 的功能。JCliff 集合支持直接从 Ansible 配置几个应用服务器子系统。](https://github.com/wildfly-extras/ansible_collections_jcliff/releases/tag/v0.0.2)

在第 1 部分中，我们将主要关注基础工作，并讨论在 Ansible 中使用 JCliff 所需的所有步骤。一旦正确安装，我们将使用 JCliff 来配置 WildFly 的`system_props`子系统，这允许我们在 WildFly 的服务器配置中声明系统变量。一旦我们有了这个基础，我们将在[第二部分](https://developers.redhat.com/blog/2020/12/03/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-2/)和[第三部分](https://developers.redhat.com/blog/2020/12/21/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-3/)中开始探索更有趣的配置。

**注意**:关于 [Ansible collections](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html) 的更多信息，请参见 Ansible 文档。

## 使用 Ansible 微调 WildFly 服务器配置

在我们开始制定可行的行动手册之前，让我们讨论一下我们想要达到的目标。我们的用例是微调 WildFly 服务器配置，这意味着超越 Ansible 的原语可以为该软件做的事情。

使用 Ansible 的内置模块很容易实现 WildFly 安装的自动化。您可以使用包管理器来安装任何必要的依赖项，创建所需的目录结构，设置配置文件，等等。然而，你不能轻易地微调 WildFly 服务器的配置。为服务器的配置提供一个可翻译的模板文件的通常策略是不合适的。许多服务器配置位于主配置文件中(`standalone.xml`或`standalone-full.xml`)。服务器在运行时频繁地访问这个文件，同时改变文件内容。因此，主配置文件不是[模板化](https://docs.ansible.com/ansible/latest/modules/template_module.html)的好目标。

我们的目标是为 WildFly 服务器提供一个微调的配置。在 Ansible playbook 中，我们希望有一种方法来定义服务器所需的状态。我们还希望确保 Ansible 能够监控这个状态，并发现对它的任何更改。这就是 JCliff ansi ble 集合的用武之地。

## JCliff 的 Ansible 集合是什么？

JCliff 是一个命令行界面(CLI ),让我们可以动态地改变 WildFly 服务器的状态。JCliff 使用 WildFly 的 JBoss CLI 在运行时更新服务器配置。该工具随应用服务器一起提供。一旦服务器处理了使用 JBoss CLI 工具发送的请求，它将确保记录这些更改，并相应地更新主配置文件`standalone.xml`。

JCliff ( `wildfly.jcliff`)的新 [Ansible 集合以 Ansible 模块的形式提供了集成。Ansible 使用 JCliff 集成来验证 WildFly 服务器配置是否处于正确的状态。如果有任何差异，它还使用 JCliff 来更新服务器配置。JCliff 构建了用于这些目的的 JBoss CLI 查询。JCliff 还有助于 Ansible 保持*幂等*，这意味着只有当配置处于正确的状态*而不是*时，才会应用更改。](https://galaxy.ansible.com/wildfly/jcliff)

JCliff 让我们直接在 Ansible 剧本中表达我们的需求。为此，我们将使用`jcliff:`模块，它是由`wildfly.jcliff`集合提供的。

**注**:如果你想了解更多关于 JCliff 的知识，请看文章 [*使用 JCliff*](https://developers.redhat.com/blog/2019/11/06/managing-jboss-eap-wildfly-using-jcliff/) 管理 JBoss EAP/WildFly。本系列以那篇文章为基础，向您展示如何使用 JCliff 新的 Ansible 集合格式在 Ansible 中打包扩展。非常感谢我的合著者 [Andrew Block、](https://developers.redhat.com/blog/author/ablock/)对本系列的评论。

## 演示的先决条件

在开始演示之前，让我们确保已经具备了所有的先决条件:

*   红帽企业版 Linux(RHEL)、CentOS 或 Fedora 操作系统。
*   用于 JDK 8 或更高版本的 Java 虚拟机。
*   Ansible (我们推荐 Ansible 2.9，但是旧的或者更新的版本应该可以)。
*   [WildFly 19](https://www.wildfly.org/downloads/) 或更高版本，应该安装并运行在系统上。

**注意**:JCliff 的 Ansible 集合包括一个自动安装 JCL IFF 的角色。目前只对 Linux 和 macOS 提供完全支持，但是支持 Windows 的工作正在进行中。

## 可行的战术手册

对于我们的演示，我们将从一个最小的、功能可行的剧本开始。为了使演示简单且易于重现，我们在本地系统上运行 Ansible。

```
---

- hosts: localhost

  gather_facts: true

  vars:

  tasks:

```

让我们验证 Ansible 是否正确安装在控制节点上，以及行动手册是否成功完成:

```
$ ansible-playbook playbook.yml

[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ***********************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************

ok: [localhost]

PLAY RECAP *****************************************************************************************************************************

localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

到目前为止，一切正常。注意，我们将`gather_facts`属性设置为`true`。此设置允许 Ansible 收集有关系统的信息。很快，Ansible 将使用这些信息来安装 JCliff。

**注意**:虽然收集事实并不是 JCliff Ansible 集合的硬性要求，但是如果您选择禁用该设置，您可能需要自己提供额外的变量。

## 正在为 JCliff 安装 Ansible 集合

我们准备为 JCliff 安装 Ansible 集合。首先，我们将使用`ansible-galaxy`工具(默认情况下包含在 Ansible 中)来安装`wildfly.jcliff`模块:

```
# ansible-galaxy collection install wildfly.jcliff

Process install dependency map

Starting collection install process

Installing 'wildfly.jcliff:0.0.2' to '/root/.ansible/collections/ansible_collections/wildfly/jcliff'

```

### 找到`JBOSS_HOME`变量

正如我前面提到的，JCliff Ansible 集合带有一个自动安装 JCliff 的角色。我们所要做的就是将所需的说明添加到我们可行的行动手册中。特别是，JCliff 需要`JBOSS_HOME`环境变量来定位目标系统上的 WildFly 服务器。在继续之前，让我们从这个环境变量中检索服务器位置:

```
---

- hosts: localhost

  gather_facts: true

  vars:

    jboss_home: "{{ lookup('env','JBOSS_HOME') }}"

  collections:

    - wildfly.jcliff

  roles:

    - jcliff

  tasks:

```

请注意以下几点:

*   变量`jboss_home`是用名为`JBOSS_HOME`的环境变量的值定义的，这确保了它们之间没有差异。
*   这个剧本需要`wildfly.jcliff`集合，我们刚刚在本地安装了这个集合。
*   Ansible 剧本从`wildfly.jcliff`集合中导入`jcliff`角色。这个角色负责在系统上安装 JCliff 本身。

接下来，我们将使用安装 JCliff 所需的说明和信息运行 Ansible playbook。

### 运行行动手册以安装 JCliff CLI

运行 Ansible 剧本导入`jcliff`角色并执行以下任务来安装 JCliff CLI:

```
# ansible-playbook playbook.yml

[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [localhost] ***********************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Collect Supported Operating Systems] *********************************************************************

ok: [localhost] => (item={u'key': u'homebrew', u'value': [u'MacOSX']})

ok: [localhost] => (item={u'key': u'rpm', u'value': [u'Fedora', u'CentOS', u'RedHat']})

TASK [wildfly.jcliff.jcliff : Verify supported Operating Systems] **********************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Install JCliff using HomeBrew] ***************************************************************************

skipping: [localhost]

TASK [wildfly.jcliff.jcliff : Install JCliff using RPM] ********************************************************************************

included: /root/.ansible/collections/ansible_collections/wildfly/jcliff/roles/jcliff/tasks/install_rpm.yml for localhost

TASK [wildfly.jcliff.jcliff : Add JCliff Yum Repository (RedHat)] **********************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Test if package jcliff is already installed] *************************************************************

fatal: [localhost]: FAILED! => {"changed": false, "cmd": ["rpm", "-q", "jcliff"], "delta": "0:00:00.489702", "end": "2020-08-17 08:54:34.190992", "msg": "non-zero return code", "rc": 1, "start": "2020-08-17 08:54:33.701290", "stderr": "", "stderr_lines": [], "stdout": "package jcliff is not installed", "stdout_lines": ["package jcliff is not installed"]}

TASK [wildfly.jcliff.jcliff : Ensure JCliff is installed] ******************************************************************************

changed: [localhost]

TASK [wildfly.jcliff.jcliff : Install Jcliff using standalone binary] ******************************************************************

skipping: [localhost]

PLAY RECAP *****************************************************************************************************************************

localhost                  : ok=6    changed=1    unreachable=0    failed=0    skipped=2    rescued=1    ignored=0

```

剧本扮演了`jcliff`的角色。新角色注意到系统上没有安装 JCliff，并安装了必要的软件。

### 验证 JCliff 安装

现在，让我们验证是否安装了 JCliff:

```
# jcliff --version

No JBOSS_HOME provided, aborting...

```

已经安装了 JCliff，但是(正如我前面提到的)，它需要我们定义`JBOSS_HOME`变量才能起作用:

```
$ export JBOSS_HOME=/path/to/wildfly/home
```

导出环境变量后，尝试重新运行命令:

```
# jcliff

Jcliff version 2.12.5

Usage:

    jcliff [options] file(s)

where the options are:

  --cli=Path : jboss-cli.sh. Defaults to

               /usr/share/jbossas/bin/jboss-cli.sh

  --controller=host       : EAP6 host. Defaults to localhost.

  --user=username         : EAP6 admin user name

  --password=pwd          : EAP6 admin password

  --ruledir=Path          : Location of jcliff rules.

  --noop                  : Read-only mode

  --json                  : Use json to parse input files

  -v                      : Verbose output

  --timeout=timeout       : Command timeout in milliseconds

  --output=Path           : Log output file

  --reload                : Reload after each subsystem configuration if required

  --waitport=waitport     : Wait this many seconds for the port to be opened

  --nobatch               : Don't use batch mode of jboss-cli

  --redeploy              : Redeploy all apps

  --reconnect-delay=delay : Wait this many milliseconds after a :reload for the server to restart

  --leavetmp              : Don't erase temp files

  --pre=str               : Prepend str to all commands (can be used for domain mode support)

```

JCliff 现在已经安装完毕，可以完全运行了。

### 验证幂等性

在设置完成之前，还有最后一件事要做。我们必须确保我们的可行剧本是幂等的。在这种情况下，这意味着如果系统已经处于适当的状态，就不会再次执行行动手册中描述的任务。要检查这一点，请再次运行行动手册，并验证不再应用任何更改:

```
# ansible-playbook  playbook.yml

PLAY [localhost] *****************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Collect Supported Operating Systems] ***************************************************************************************************************************

ok: [localhost] => (item={u'key': u'homebrew', u'value': [u'MacOSX']})

ok: [localhost] => (item={u'key': u'rpm', u'value': [u'Fedora', u'CentOS', u'RedHat']})

TASK [wildfly.jcliff.jcliff : Verify supported Operating Systems] ****************************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Install JCliff using HomeBrew] *********************************************************************************************************************************

skipping: [localhost]

TASK [wildfly.jcliff.jcliff : Install JCliff using RPM] **************************************************************************************************************************************

included: /root/.ansible/collections/ansible_collections/wildfly/jcliff/roles/jcliff/tasks/install_rpm.yml for localhost

TASK [wildfly.jcliff.jcliff : Add JCliff Yum Repository (RedHat)] ****************************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Test if package jcliff is already installed] *******************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Install Jcliff using standalone binary] ************************************************************************************************************************

skipping: [localhost]

PLAY RECAP ***********************************************************************************************************************************************************************************

localhost                  : ok=6    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

```

我们已经完成了将 Ansible 剧本与 JCliff 集成的设置。现在，我们准备开始使用`jcliff:`模块来微调我们的 WildFly 服务器配置。

## 在 WildFly 服务器配置中定义系统变量

我们将从使用 JCliff 向 WildFly 的服务器配置添加系统变量开始。这不是最复杂的功能，但却是确认一切正常的好方法:

```
---

- hosts: localhost

  gather_facts: true

  vars:

    jboss_home: "{{ lookup('env','JBOSS_HOME') }}"

  collections:

    - wildfly.jcliff

  roles:

    - jcliff

  tasks:

```

到目前为止，配置很简单。JCliff CLI 只需要 WildFly 服务器主目录的路径，这是我们在`jboss_home`变量中定义的。定位 home 变量后，JCliff 使用 WildFly 提供的脚本(`${JBOSS_HOME}/bin/jboss-cli.sh`)与 WildFly 服务器通信。

现在，我们要开始我们的定制配置。大多数 WildFly 配置被定义为子系统，JCliff 有这些子系统的列表。我们将从简单的东西开始:`system_props`子系统，我们将使用它来声明 WildFly 服务器配置中的系统变量:

```
    - jcliff:

        wfly_home: "{{ jboss_home }}"

        subsystems:

          - system_props:

              - name: jcliff.enabled

                value: 'enabled.plus'

```

重新运行 Ansible 剧本，看看会发生什么:

```
# ansible-playbook playbook.yml

PLAY [localhost] *****************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Collect Supported Operating Systems] ***************************************************************************************************************************

ok: [localhost] => (item={u'key': u'homebrew', u'value': [u'MacOSX']})

ok: [localhost] => (item={u'key': u'rpm', u'value': [u'Fedora', u'CentOS', u'RedHat']})

TASK [wildfly.jcliff.jcliff : Verify supported Operating Systems] ****************************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Install JCliff using HomeBrew] *********************************************************************************************************************************

skipping: [localhost]

TASK [wildfly.jcliff.jcliff : Install JCliff using RPM] **************************************************************************************************************************************

included: /root/.ansible/collections/ansible_collections/wildfly/jcliff/roles/jcliff/tasks/install_rpm.yml for localhost

TASK [wildfly.jcliff.jcliff : Add JCliff Yum Repository (RedHat)] ****************************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Test if package jcliff is already installed] *******************************************************************************************************************

ok: [localhost]

TASK [wildfly.jcliff.jcliff : Install Jcliff using standalone binary] ************************************************************************************************************************

skipping: [localhost]

TASK [jcliff] ********************************************************************************************************************************************************************************

changed: [localhost]

PLAY RECAP ***********************************************************************************************************************************************************************************

localhost                  : ok=7    changed=1    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

```

一切似乎都很顺利。Ansible 注意到我们定义的变量在服务器的初始配置中缺失，并添加了它们。所有这些都归功于 JCliff 与服务器的通信(通过 JBoss CLI)。

要确认变量是在 WildFly 配置中定义的，请运行以下 JBoss CLI 查询:

```
# ${JBOSS_HOME}/bin/jboss-cli.sh --connect --command='/system-property=jcliff.enabled:read-resource'

{

    "outcome" => "success",

    "result" => {"value" => "enabled.plus"}

}

```

我们来彻底验证一下 JCliff 的一个承诺。我们之前说过 WildFly 服务器的 XML 配置会自动更新。这是已经确认的:

```
...

 </extensions>

    <system-properties>

        <property name="jcliff.enabled" value="enabled.plus"/>

    </system-properties>

    <management>

...

```

一切都在按预期运行。JCliff 可以与 WildFly 服务器通信，并且可以更新服务器配置。

## 结论

我们已经完成了基本设置。在第 2 部分中，我们将深入探讨 JCliff 的 Ansible 集合所提供的特性。我们将讨论如何部署新的 JDBC 驱动程序以及定义新的数据源。我们还将在应用服务器内部部署应用程序。

*Last updated: December 23, 2020*