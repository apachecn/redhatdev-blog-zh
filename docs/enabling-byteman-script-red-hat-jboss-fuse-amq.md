# 使用 Red Hat JBoss Fuse 和 AMQ 启用 Byteman 脚本-第 1 部分

> 原文：<https://developers.redhat.com/blog/2018/01/02/enabling-byteman-script-red-hat-jboss-fuse-amq>

在生产或客户环境中，并不总是能够通过查看日志来识别问题，也不总是能够使用集成开发环境( **IDE)** 和远程调试端口来设置远程调试。这些问题通常是特定于环境的，无法重现。在这些情况下，拥有**字节曼**脚本可以帮助识别问题，而无需实际的代码更改。每当调用某个 java 类或逻辑时， **byteman** 脚本也会按照 **byteman** 脚本中定义的类和方法被调用。

以下是要遵循的步骤:

1.下载 **Byteman** 二进制[这里](http://byteman.jboss.org/downloads.html)。我下载了 3.0.10 版本:[http://downloads . JBoss . org/byte man/3 . 0 . 10/byte man-download-3 . 0 . 10-bin . zip](http://downloads.jboss.org/byteman/3.0.10/byteman-download-3.0.10-bin.zip)。

2.在**红帽 JBoss 保险丝**存在的同一台机器中提取。

3.提取之后，创建一个文本文件(**byte man**script)*script . btm*(文件位于*/path/to/byte man-download-3 . 0 . 10*)。这里 *byteman-download-3.0.10* 是解压二进制后创建的文件夹。

4.该脚本的内容将是:

```
RULE check authpassword
CLASS org.apache.karaf.shell.ssh.KarafJaasAuthenticator
METHOD authenticate(java.lang.String, java.lang.String, org.apache.sshd.server.session.ServerSession)
AT ENTRY
IF true
DO 
traceOpen("file.out", "byteman.log");
traceln("file.out"," password: "+ $2);
ENDRULE

```

在这里指出注意:

*   我们要分析 **KarafJaasAuthenticator** 类的**认证**方法。我们想检查第二个参数的值，即密码。
*   我们正在使用 **traceOpen** 以便 **traceln** 日志最终被写入 byteman.log。如果我们不使用 **traceOpen，**那么 **traceln** 日志将被打印在 **Karaf 的**终端中，因此我们可能会丢失日志。

5.在**红帽 JBoss Fuse 中，**我们必须编辑文件*$ { karaf . home }/etc/config . properties*并修改属性**org . OSGi . framework . boot delegation**，以便它也包括包' **org.jboss.byteman.*** '，如下所示:

```
org.osgi.framework.bootdelegation=org.apache.karaf.jaas.boot,sun.*,com.sun.*,javax.transaction,javax.transaction.*,org.apache.xalan.processor,org.apache.xpath.jaxp,org.apache.xml.dtm.ref,org.apache.xerces.jaxp.datatype,org.apache.xerces.stax,org.apache.xerces.parsers,org.apache.xerces.jaxp,org.apache.xerces.jaxp.validation,org.apache.xerces.dom,org.jboss.byteman.*
```

6.现在编辑 **${karaf.home}/bin/setenv** 文件并包含 **JAVA_OPTS** jvm 参数，如下所示。这个参数指的是 **byteman** jar 和**byte man**script*script . btm*。

```
export JAVA_OPTS="-javaagent:/path/to/byteman-download-3.0.10/lib/byteman.jar=script:/path/to/byteman-download-3.0.10/script.btm,boot:/path/to/byteman-download-3.0.10/lib/byteman.jar"
```

7.这个脚本的目的是检查为身份验证输入的密码是否在代码中正确传递。类似地，可能还有其他的用例需要检查代码的执行。

实际调用登录的 java 类是:**org . Apache . karaf . shell . ssh . karafjaasauthenticator**
调用的方法: **public boolean authenticate(最终字符串用户名，最终字符串密码，最终服务器会话会话)**

这些细节我们可以在解决问题时获得。我在调试级日志中发现了这个类。所以故障排除总是从日志开始。

8.为了测试，首先使用启动脚本启动 **Red Hat JBoss Fuse** 。

```
[cpandey@cpandey bin]$ pwd
/path/to/jboss-fuse-6.3.0.redhat-310/bin
[cpandey@cpandey bin]$ ./start
```

9.现在运行客户端脚本来访问 karaf 终端。我们输入的密码是“错误密码”。

```
[cpandey@cpandey bin]$ ./client -u admin
Logging in as admin
Password: 
Password:
```

10.现在检查我们在脚本文件 *script.btm* 中配置的 byteman.log。我们应该在日志中输入密码。

```
[cpandey@cpandey jboss-fuse-6.3.0.redhat-310]$ pwd
/path/to/jboss-fuse-6.3.0.redhat-310

[cpandey@cpandey jboss-fuse-6.3.0.redhat-310]$ tail -f byteman.log 
 password: wrongpassword
```

上面的例子是一个真实的用例。如果我们想从日志不足或不可用的代码中获取信息，我们可以解决其他问题。这也已经在织物环境中进行了测试。

就是这样。感谢阅读！

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: January 12, 2018*