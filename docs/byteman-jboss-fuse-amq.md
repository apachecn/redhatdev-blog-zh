# 使用 Red Hat JBoss Fuse 和 AMQ 启用 Byteman 脚本——第 2 部分

> 原文：<https://developers.redhat.com/blog/2018/01/18/byteman-jboss-fuse-amq>

在我的上一篇文章中，[使用红帽 JBoss Fuse 和 AMQ 启用 Byteman 脚本-第 1 部分](https://developers.redhat.com/blog/2018/01/02/enabling-byteman-script-red-hat-jboss-fuse-amq/)，我们找到了一个使用红帽 JBoss Fuse 或红帽 JBoss AMQ 的 Byteman 脚本的基本用例。但是，日志文件是单独生成的，只能进行有限的操作。在本文中，我将向您展示如何使用 Java 助手类。通过使用 Java，我们获得了查看或修改内容的高级操作。此外，使用 java.util.logging 允许我们将语句记录到 fuse.log，避免创建任何其他日志文件。

除了 Helper 类之外，大多数步骤与我的上一篇文章相同。为了完整，我包括了所有的步骤。

1.从 downloads.jboss.org/byteman 的[下载 **Byteman** binary。我下载](http://byteman.jboss.org/downloads.html)[3 . 0 . 10 版](http://downloads.jboss.org/byteman/3.0.10/byteman-download-3.0.10-bin.zip)。

2.在**红帽 JBoss 保险丝**存在的同一台机器中提取。

3.提取之后，在位置`/path/to/byteman-download-3.0.10`创建一个文本文件(byteman 脚本)`script.btm`文件。这里的`byteman-download-3.0.10`是在解压二进制文件后创建的文件夹中。该脚本的内容:

```
HELPER com.myexample.helper.BytemanHelper
RULE check authpassword
CLASS org.apache.karaf.management.JaasAuthenticator
METHOD authenticate(java.lang.Object)
BIND msg = $1
AT ENTRY
IF TRUE
DO
helperMethod(msg);
ENDRULE

```

这里要注意的几点。

*   我们使用的是一个助手类:
    `com.myexample.helper.BytemanHelper`。
*   这个类是一个简单的 java 类，它扩展了:
    `org.jboss.byteman.rule.helper.Helper`。
*   我们在这个类中有一个方法“helperMethod ”,我们使用这个脚本执行它。

4.这里我们要做的是有类:
`org.apache.karaf.management.JaasAuthenticator`
有方法:
`public Subject authenticate(Object credentials)`。
这个方法从 credentials 参数(一个对象类型)中提取用户名和密码。在此方法中，凭据是从字符串类型中获取的。

```
final String[] params = (String[]) credentials;
```

在我的上一篇文章中，我没有找到仅使用 Byteman 脚本实现这一点的方法。相反，我使用了一个助手类，这样使用 Java 提供的工具，我可以很容易地修改或获取细节。

5.下面是 BytemanHelper 类的内容。这里我们有一个 helperMethod，它是通过我们在上面创建的 Byteman 脚本' script.btm '调用的。这个方法从 Java 对象中获取一个字符串数组并记录下来。

```
package com.myexample.helper;

import java.util.logging.Logger;
import org.jboss.byteman.rule.Rule;
import org.jboss.byteman.rule.helper.Helper;

public class BytemanHelper
extends Helper
{
Logger logger;

protected BytemanHelper(Rule rule)
{
super(rule);
this.logger = Logger.getLogger(rule.getTriggerClass() + ":" + rule.getTriggerMethod());
}

public void helperMethod(Object object)
{
String[] params = (String[])object;
this.logger.info("Byteman fetched username: " + params[0]);
this.logger.info("Byteman fetched Password: " + params[1]);
}
}
```

6.现在我们必须用这个类和完整的包结构创建一个 Jar 文件。您可以将项目作为 Jar 从您最喜欢的 IDE 中导出，我使用的是 Eclipse。或者，您也可以从终端轻松创建它。转到项目中包的位置，我在我的包`com.myexample.helper`所在的 src 文件夹中执行了下面的命令。

```
[cpandey@cpandey src]$ jar cvf BytemanFuseHelper.jar .
```

7.现在编辑`${karaf.home}/etc/config.properties`并修改`org.osgi.framework.bootdelegation`属性，使其也包括`org.jboss.byteman package`:

```
org.osgi.framework.bootdelegation=org.apache.karaf.jaas.boot,org.apache.karaf.management.boot,sun.*,com.sun.*,javax.transaction,javax.transaction.*,javax.xml.crypto,javax.xml.crypto.*,org.apache.xerces.*,org.bouncycastle.*,com.ibm.security.*,org.apache.xalan.processor, org.jboss.byteman.*
```

8.现在编辑`${karaf.home}/bin/setenv`(或者 windows 的`setenv.bat`)并包含如下的`JAVA_OPTS` jvm 参数。这个参数指的是 byteman jar 和 byteman 脚本`script.btm`。用解压 byteman zip 文件的路径替换`/path/to`。

```
export JAVA_OPTS="-javaagent:/path/to/byteman-download-3.0.10/lib/byteman.jar=script:/path/to/byteman-download-3.0.10/script.btm,boot:/path/to/byteman-download-3.0.10/lib/byteman.jar,sys:/path/to/BytemanFuseHelper.jar -Dorg.jboss.byteman.verbose"
```

注意:您必须添加助手`sys:/path/to/BytemanFuseHelper.jar` jar 的路径以及前缀`sys:`。

9.要进行测试，首先使用启动脚本启动 **Red Hat JBoss Fuse** :

```
[cpandey@cpandey bin]$ pwd
/path/to/jboss-fuse-6.3.0.redhat-310/bin
[cpandey@cpandey bin]$ ./start
```

10.现在启动或停止子容器，在我们修改的根容器中检查`fuse.log`。

11.现在检查`fuse.log`。我们应该在日志中看到输入的密码。我们现在可以检查用户和密码是否已更新。

```
2018-01-01 12:22:33,429 | INFO | on(12)-127.0.0.1 | JaasAuthenticator:authenticate | - - | Byteman fetched username: admin
2018-01-01 12:22:33,429 | INFO | on(12)-127.0.0.1 | JaasAuthenticator:authenticate | - - | Byteman fetched Password: cs1
```

就这些，谢谢你的阅读。我希望您现在理解我们为什么需要一个助手类，以及它与我上一篇文章中的方法有什么不同和更好。

*Last updated: January 19, 2018*