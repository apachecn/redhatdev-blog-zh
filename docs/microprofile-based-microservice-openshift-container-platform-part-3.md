# OpenShift 容器平台上基于微概要文件的微服务——第三部分

> 原文：<https://developers.redhat.com/blog/2017/10/20/microprofile-based-microservice-openshift-container-platform-part-3>

# 通过基于微概要文件的微服务创建、关联和使用 JBoss 数据网格(具有持久性)

在这篇博文中，我将介绍如何创建、填充 JBoss 数据网格，并将其与基于 MicroProfile 的微服务相关联。我还将介绍如何修改微服务，以便它能够利用 JBoss 数据网格(JDG)。

这是“OpenShift 容器平台上基于微概要文件的微服务-第二部分”的继续。创建，关联和使用一个数据库与一个基于 MicroProfile 的微服务“博客文章和它作为 假设你已经完成了它的所有步骤。如果您尚未完成第 2 部分，请现在完成。

![](img/fa81eea6efb1c3d5cb29d484129749a0.png)

## 步伐

### 将和 JDG 的 JDG 图像流和持久模板添加到 OpenShift 容器平台

模板描述了一组对象，这些对象可以被参数化和处理，以生成一个对象列表，供 OpenShift 容器平台创建。有一套 OpenShift 中间件模板样本，可以在找到

[https://github.com/jboss-openshift/application-templates](https://github.com/jboss-openshift/application-templates)

我们将使用其中一个模板，即“JDG 7.1 + postgreSQL 持久化”模板。

我的 OpenShift 容器平台没有 JDG 的图像流，所以我们需要在安装模板之前安装它。要获取 JDG 的图像流，请访问:

[https://raw . githubusercontent . com/JBoss-open shift/application-templates/master/JBoss-image-streams . JSON](https://raw.githubusercontent.com/jboss-openshift/application-templates/master/jboss-image-streams.json)

(非 raw 版本在此:[https://github . com/JBoss-open shift/application-templates/blob/master/JBoss-image-streams . JSON](https://github.com/jboss-openshift/application-templates/blob/master/jboss-image-streams.json))

上面的链接包含了许多 JBoss 中间件的图像流，但我们只想要 JDG 的图像流。在原始文件中找到 JDG 的部分(搜索字符串“jboss-datagrid71-openshift”以找到 JDG 7.1 的块)，并将 JDG 图像流块复制并粘贴到一个本地文件，您可以将该文件命名为“JBoss-datagrid-image-stream . JSON”。如果您不想在 raw 文件中搜索该块，您可以将以下文本复制并粘贴到您的本地文件"JBoss-datagrid-image-stream . JSON"(我将它放在我的$HOME 目录中):

 **{*
*【种类】:" ImageStream "，*
*【API version】:" v1 "，*
*【元数据】:{*
*【名称】:" jboss-datagrid71-openshift "，** 
*【规格】:{*
*【标签】:*
*{*
*【名称】:【1.0-8】，*
*【注释】:{* 、
*" icon class ":" icon-jboss "、*
*" tags ":" datagrid，JBoss，xpaas "、*
*" supports ":" datagrid:7.1，xpaas:1.0 "，*
*" version ":"*
*【名称】:" registry . access . red hat . com/JBoss-datagrid-7/datagrid 71-open shift "*
*}*
*}*
*】*

接下来我们需要安装“JDG 7.1 + postgreSQL 持久化”模板，可以在这里找到:

[https://raw . githubusercontent . com/JBoss-open shift/application-templates/master/datagrid/datagrid 71-PostgreSQL-persistent . JSON](https://raw.githubusercontent.com/jboss-openshift/application-templates/master/datagrid/datagrid71-postgresql-persistent.json)

(非 raw 版本可以在这里找到:[https://github . com/JBoss-open shift/application-templates/blob/master/datagrid/datagrid 71-PostgreSQL-persistent . JSON](https://github.com/jboss-openshift/application-templates/blob/master/datagrid/datagrid71-postgresql-persistent.json))

将上面文件的全部内容复制粘贴到一个名为“datagrid 71-PostgreSQL-persistent . JSON”的本地文件中(我放在我的$HOME 目录下)。**注意:**在粘贴的文本中，请确保模板的 datagrid 组件的 ImageStreamTag 与 JDG 图像流的版本号相匹配。在粘贴的文本中，搜索文本中的字符串“jboss-datagrid71-openshift:”并将字符串从“jboss-datagrid71-openshift:TP”更改为“JBoss-datagrid 71-open shift:1.0-8”。

现在，我们需要在 OpenShift 集群中创建 JDG 图像流和模板，它应该已经启动并运行(如果没有，从“终端”窗口输入“oc cluster up”)。为此，首先打开一个“终端”窗口，通过输入:登录到您的 OpenShift 集群

*$ oc 登录-u 系统:管理员*

![](img/e12e74ab4471768b1dae28b022d216f0.png)

确保你当前的项目是“我的项目”。如果没有，请在命令提示符下输入“oc project myproject”。

此时，我们通过输入以下命令创建 JDG 图像流:

*$ oc create-n open shift-f JBoss-datagrid-image-stream . JSON*

![](img/47196002f0758db39c5278ffd3ad7006.png)

然后，我们需要通过输入:创建“JDG 7.1+postgreSQL 持久化”模板

*$ oc create-n open shift-f datagrid 71-PostgreSQL-persistent . JSON*

![](img/ac65e83c78997f4d997e366211431957.png)

### 在您的项目中创建一个具有持久性模板的 JDG 实例

该模板需要创建一个 serviceaccount，添加一个角色策略，并创建 SSL 和 JGroups 密钥库。为了方便起见，我将所有这些命令放在一个名为“createSecrets.ksh”的 Korn shell 脚本中，其内容如下所示:

*#！/bin/ksh*
*oc 创建 service account datagrid-service-account*
*oc policy add-role-to-user view system:service account:$(oc project-q):datagrid-service-account*
*OpenSSL req-new-new key RSA:4096-x509-keyut xpaas . key-out xpaas . CRT-days 365-subj "/" -CA xpaas . CRT-CAkey xpaas . key-in datagrid . CSR-out datagrid . CRT-days 365-cacreatesserial*
*keytool-import-file xpaas . CRT-alias xpaas . CA-keystore datagrid-https . jks*
*keytool-import-file datagrid . CRT-alias datagrid-https-key-keystore datagrid-https . jks*

你可以剪切并粘贴上面的命令，把它们放在一个本地文件中并执行它，或者你可以在已经打开的同一个“终端”窗口中一次输入一个命令。我选择如下执行 shell 脚本(我为所有密码短语/密码提示输入了相同的密码):

*$。/create secrets . ksh*

![](img/966221b930865a348d8f756b521175d1.png)

...

![](img/25a6693d7b604af19b7f571e99c49f7c.png)

...

![](img/b42ece9b9e63c9c9710a97ce49b5ac22.png)

这里是上面 Korn shell 脚本执行的命令列表，并附有每个命令的简短说明:

1.  创建一个名为“数据网格-服务-帐户”的服务帐户:

*$ oc 创建服务帐户数据网格-服务-帐户*

2.  向服务帐户添加查看角色:

*$ oc 策略添加-角色到用户视图系统:service account:$(oc project-q):datagrid-service-account*

3.  模板需要一个 SSL 密钥库和一个 JGroups 密钥库。 生成一个 CA 证书:

*$ OpenSSL req-new-new key RSA:4096-x509-key out xpaas . key-out xpaas . CRT-days 365-subj "/CN = xpaas-datagrid-demo . ca "*

4.  为 SSL 密钥库生成证书:

*$keytool-genkey pair-keyalg RSA-keysize 2048-dname " CN = secure-datagrid-my project . open shift 32 . example . com "-别名 datagrid-https-key-keystore datagrid-https . jks*

5.  为 SSL 密钥库生成证书签名请求:

*$keytool-certreq-keyalg RSA-alias datagrid-https-key-keystore datagrid-https . jks-file datagrid . CSR*

6.  用 CA 证书对证书签名请求进行签名:

*$OpenSSL x509-req-CA xpaas . CRT-CAkey xpaas . key-in datagrid . CSR-out datagrid . CRT-days 365-cacreatesserial*

7.  将 CA 导入 SSL 密钥库:

*$keytool-导入-文件 xpaas . CRT-别名 xpaas . ca-密钥库 datagrid-https.jks*

8.  将已签名的证书签名请求导入 SSL 密钥库:

*【$】keytool-import-file datagrid . CRT-alias datagrid-https-key-keystore datagrid-https . jks*

9.  将 CA 导入新的信任库密钥库:

*$keytool-导入-文件 xpaas . CRT-别名 xpaas . ca-keystore keystore . jks*

10.  为 JGroups 密钥库生成安全密钥:

*$keytool-gens eckey-alias j groups-storetype JCE ks-keystore j groups . JCE ks*

11.  使用 JGroups 密钥库、SSL 密钥库和信任库创建密钥库秘密。这个示例中包含了信任库，因为它是对示例 SSL 密钥库进行签名所必需的。

*$oc secret new datagrid-j group-secret j groups . JCE ks*

*$ oc 秘密新建数据网格-app-秘密数据网格-https.jks keystore.jks*

12.  将秘密链接到之前创建的服务帐户:

*$oc secrets link datagrid-service-account datagrid-j group-secret datagrid-app-secret*

此时，我们需要使用上面的模板创建一个 pod。进入 OpenShift 控制台([https://127 . 0 . 0 . 1:8443](https://127.0.0.1:8443))以开发者/开发者身份登录。

![](img/794d968236aba15088531a02c674bfac.png)

点击“我的项目”打开您的项目。

![](img/bbe4f4a035760b886833b0aaa523e766.png)

点击“添加到项目”按钮。

![](img/3a1023f0e8633752d85a8b55c7e793ac.png)

在“添加到项目”窗口中，在搜索栏中输入“jdg”,“从 web 框架、数据库和其他组件中选择以将内容添加到您的项目中”

![](img/ca521f551fbae8146823956011074c24.png)

点击标题为“红帽 JBoss 数据网格 7.1 + PostgreSQL(持久化 https)”的模板的“选择”按钮。这将打开模板设置屏幕。

![](img/d550d38c2066d1f5fd8d2b2fecdf3b1d.png)

在模板创建屏幕中，确保在指示的字段中填充以下值(保留其他预填充的字段值):

| **字段名** | **字段值** |
| 应用名称 | 数据网格应用 |
| 服务器密钥库秘密名称 | 数据网格-应用程序-秘密 |
| 服务器密钥库文件名 | keystore.jks |
| 数据库名称 | mydb |
| 数据库用户名 | dbuser |
| 数据库密码 | 数据库密码 |
| Infinispan 连接器 | 休息 |
| Memcached 缓存名称 | 默认 |
| JGroups 秘密名称 | 数据网格-应用程序-秘密 |
| JGroups 密钥库文件名 | jgroups.jceks |
| PostgreSQL 图像流标签 | 9.5 |

确认所有字段都已设置为其指示值后，点击页面底部的创建按钮。

![](img/669b8a2ff54eef17b701c04ecf83a0c7.png)

该模板将创建一个 JDG 实例(实际上是两个:一个安全的，一个不安全的)和一个相应的 postgresql 数据库，该数据库带有一个表(其名称将包含后缀“default”)，它将内存中的键和值保存到该表中。 模板成功创建后，您将看到以下屏幕:

![](img/fb1126d8829df2a8d9b5ac3d634c5a23.png)

点击“继续概述”。这将把您带到控制台上的概览屏幕，在这里您将看到您的 JDG 和 Postgresql pods 已启动并正在运行。

![](img/7c1f5a721ed7f77432024843a7b8cda6.png)

要检查自动创建的 postgresql 工件，请在“datagrid-app-postgresql”窗格中单击标有“1 pod”的圆圈，进入您的 Postgresql 窗格。

![](img/94a1448e5b1b2c1c445a7965b02cade9.png)

点击“终端”选项卡。

![](img/3f7e6d63c86a755137f44577da432245.png)

在 pod 的终端窗口中，输入:

*$ psql -U dbuser mydb*

![](img/f6c7d6267842490e660d4c89c78c3522.png)

要列出“mydb”数据库(由模板创建)中的所有表，请输入以下 psql 命令:

*mydb= >形容\ dt*

![](img/2ba87f3b0a93b0d3db171a9410b11306.png)

要查看 ispn_entry_default 表的结构，IMDG 数据将保存在该表中，请输入:

*mydb= > \d ispn_entry_default;*

![](img/4ee1722ef52e04dffc663143803276ce.png)

要查看保存在名为 ispn_entry_default 的 mydb 数据库表中的消息，请输入:

*mydb =>select * from ispn _ entry _ default；*

![](img/f91c259a900fe0f3067a17fe9ad3e30c.png)

这个表格是空的，因为我们还没有在 JDG 中输入任何数据。稍后，当我们将数据输入 JDG 时，您将执行相同的查询，以验证内存中的数据是否已经持久化到 Postgresql 中。

### 填充 JDG 内存数据网格

与这个由 3 部分组成的博客系列的第 2 部分类似，MicroProfile 微服务将在浏览器上显示 hello 消息，这些消息会在不同的名称之间迭代，但在这种情况下，这些名称将驻留在 JBoss 数据网格(JDG)中，而不是数据库中。在后台，JDG 会将数据保存到 Postgresql 数据库中，但这对于微服务是完全透明的。JDG 有一个 REST 接口，我们可以用它来填充数据。

【REST 调用 JDG 的 url 语法如下:

*< URL > /rest/ <缓存名称> / <缓存关键字>*

我们需要用我们基于微档案的微服务将要消费的人的名字来填充 JDG。从“终端”窗口，如果您没有登录到 OpenShift 集群，那么登录并确保您在“myproject”项目上:

*$ oc 登录-u 系统:管理员*

*$ oc 项目 myproject*

创建一个环境变量，该变量捕获数据网格应用程序的路由名称，如下所示:

*$ app _ path = $(oc describe route datagrid-app | grep " ^requested 主机:" | awk '{ print $3 }')*

然后输入以下命令将这五个名字插入 IMDG:

*$ curl-X PUT http://$ { APP _ PATH }/rest/default/0-d“强尼”*

*$ curl-X PUT http://$ { APP _ PATH }/rest/default/1-d《珍妮》*

*$ curl-X PUT http://$ { APP _ PATH }/rest/default/2d《比利》*

*$ curl-X PUT http://$ { APP _ PATH }/rest/default/3d《玛丽》*

*$ curl-X PUT http://$ { APP _ PATH }/rest/default/4-d“巴比”*

![](img/ccd0bf998f9a3da2d7ac14fae1fbbefb.png)

JDG 还会将这些名称保存到 postgresql 数据库中。

如果您想验证这五个条目是否被保存到持久存储中，请转到 Postgresql 窗格的“终端”窗口，并在 shell 提示符下输入:

*$ psql -U dbuser mydb*

*$ select * from ispn _ entry _ default；*

![](img/34dc69ae06b1d3b3f09b2fb656ab5094.png)

这五个名字加密在 Postgresql 数据库的 ispn_entry_default 表中。

### 修改基于微文件的微服务

所以现在，我们需要修改基于微概要文件的微服务来调用 JDG。与 JDG 通信的最快协议是飞车手罗德，但我们将使用 REST 接口，因为它最容易实现，并且不需要我们将另一个野生虫群片段包含到我们基于微文件的微服务中。事实上，我们可以去掉这个由 3 部分组成的博客系列的第 2 部分中的“数据源”部分，并将其从我们的微服务中删除。

编辑完”。java”，并用下面的代码块替换它的内容:

*包 com . MP example . mphelloworld . rest；*
*导入 Java . io . buffered reader；*
*导入 Java . io . io exception；*
*导入 Java . io . inputstreamreader；*
*导入 Java . net . http urlconnection；*
*导入 Java . net . URL；*
*导入 Java . util . concurrent . threadlocalrrandom；*
*导入 javax . enterprise . context . applicationscoped；*
*导入 Java x . ws . RS . path；*
*导入 javax . ws . RS . core . response；*
*导入 javax . ws . RS . get；*
*导入 javax . ws . RS . produce；*
*@ ApplicationScoped
*@ Path("/hello ")*
*public class HelloWorldEndpoint {*
*@ GET*
*@ Produces(" text/text*
*//取 IMDG 数据项个数的 env var 并确定 key*
*int key value；*
*String numofdataitemstr = system . getenv(" NUM _ OF _ DATA _ ITEMS ")；*
*int numOfDataItems = integer . parse int(numOfDataItemsStr)；*
*key value = threadlocalrrandom . current()。nextInt(0，numOfDataItems)；*
*//获取 REST 调用 url 的 env 变量并向其追加关键字*
*String URL CACHE NAME = system . getenv(" URL _ CACHE _ NAME ")；*
*urlCacheName+= key value；*
*试{*
*String name = get method(URL cachename)；*
*greeting = " Hello "+name+" from micro profile microservice "；*
*} catch(io exception e){*
*return response . ok(" REST 调用到微服务问题！"+ e.toString())。build()；*
*}*
*return response . ok(greeting . tostring())。build()；*
*}*
*/* **
**通过 url 中的键获取值作为 param 值的方法。*
** @ param URL server address*
** @ return String value*
** @ throws io exception*
**/*
*public String get method*
*StringBuilder StringBuilder = new StringBuilder()；*
*URL 地址=新 URL(URL server address)；*
*HttpURLConnection 连接=(HttpURLConnection)address . open connection()；*
*connection . setrequestmethod(" GET ")；*
*connection . setrequestproperty(" Content-Type "，" text/plain ")；*
*connection . setdoooutput(true)；*
*buffered reader buffered reader = new buffered reader(new InputStreamReader(connection . getinputstream()))；T248
T250connection . connect()；*
*while((line = buffered reader . readline())！= null){*
*//stringbuilder . append(line+' \ n ')；*
*stringbuilder . append(line)；*
*}*
*connection . disconnect()；*
*返回 stringbuilder . tostring()；*
*}*
*}**

编辑 pom.xml 并删除“数据源”部分。新的 pom.xml 应该是这样的:

*<?xml version="1.0" encoding="UTF-8"?>*
*<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"*
*        xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">*
* <modelVersion>4.0.0</modelVersion>*
* <groupId>com.mpexample</groupId>*
* <artifactId>mpHelloWorld</artifactId>*
* <name>WildFly Swarm Example</name>*
* <version>1.0.0</version>*
* <packaging>war</packaging>*
* <properties>*
*   <version.wildfly.swarm>2017.8.1</version.wildfly.swarm>*
*   <maven.compiler.source>1.8</maven.compiler.source>*
*   <maven.compiler.target>1.8</maven.compiler.target>*
*   <failOnMissingWebXml>false</failOnMissingWebXml>*
*   <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>*
*   <fabric8.maven.plugin.version>3.1.92</fabric8.maven.plugin.version>*
* </properties>*
* <dependencyManagement>*
*   <dependencies>*
*     <dependency>*
*       <groupId>org.wildfly.swarm</groupId>*
*       <artifactId>bom-all</artifactId>*
*       <version>${version.wildfly.swarm}</version>*
*       <scope>import</scope>*
*       <type>pom</type>*
*     </dependency>*
*   </dependencies>*
* </dependencyManagement>*
* <build>*
*   <finalName>demo</finalName>*
*   <plugins>*
*     <plugin>*
*       <groupId>io.fabric8</groupId>*
*       <artifactId>fabric8-maven-plugin</artifactId>*
*       <version>${fabric8.maven.plugin.version}</version>*
*       <executions>*
*         <execution>*
*           <goals>*
*             <goal>resource</goal>*
*           </goals>*
*         </execution>*
*       </executions>*
*       <configuration>*
*         <generator>*
*           <includes>*
*             <include>wildfly-swarm</include>*
*           </includes>*
*         </generator>*
*       </configuration>*
*     </plugin>*
*     <plugin>*
*       <groupId>org.wildfly.swarm</groupId>*
*       <artifactId>wildfly-swarm-plugin</artifactId>*
*       <version>${version.wildfly.swarm}</version>*
*       <executions>*
*         <execution>*
*           <goals>*
*             <goal>package</goal>*
*           </goals>*
*         </execution>*
*       </executions>*
*     </plugin>*
*   </plugins>*
* </build>*
* <dependencies>*
*   <!-- Java EE 7 dependency -->*
*   <dependency>*
*     <groupId>javax</groupId>*
*     <artifactId>javaee-api</artifactId>*
*     <version>7.0</version>*
*     <scope>provided</scope>*
*   </dependency>*
*   <dependency>*
*     <groupId>org.postgresql</groupId>*
*     <artifactId>postgresql</artifactId>*
*     <version>9.4-1200-jdbc41</version>*
*   </dependency>*
*   <!-- WildFly Swarm Fractions -->*
*   <dependency>*
*     <groupId>org.wildfly.swarm</groupId>*
*     <artifactId>microprofile</artifactId>*
*   </dependency>*
* </dependencies>*
*</project>*

### 验证您的部署，创建路线，并测试您的基于微配置文件的微服务

此时，随着 OCP 的启动和运行，确保您在您的 mpHelloWorld 项目目录中，打开一个终端窗口并输入:

*$ cd <您选择的 dir>/mphello world*

![](img/327a61d817e4934811b42c23249eadd7.png)

然后，通过输入:登录到正在运行的 OpenShift

*$ oc 登录 https://127 . 0 . 0 . 1:8443-u developer-p developer*

![](img/aee63fa1f5526ab2527b4303c91d86a9.png)

现在，通过输入:将您的项目构建并部署到 OpenShift

*$ mvn clean fabric 8:build fabric 8:deploy-dskiptest*

![](img/75cfa98625633384942dac2d336c063a.png)

...

![](img/fb3622a14aa9021cbc5b6a5046a0f158.png)

现在，您的 MicroProfile 微服务 pod 已经启动并在 OpenShift 容器平台上运行，您必须为它创建一个路由，以便您可以从浏览器窗口调用它。在 OpenShift 容器平台控制台上，转到项目“我的项目”，在概览屏幕上，一直滚动到底部，您会看到“mphelloworld”窗格:

![](img/0220f41838f9ee4a9aeed83fc0582f86.png)

点击图标旁边的“mphelloworld ”,该图标看起来像一个带有两个箭头的小圆圈。

![](img/57de3ca601a498e11500ab1386c97c4b.png)

点击“路线:”标签旁边的“创建路线”，接受所有默认设置。

![](img/e8510a63aec862e96cb16a299fdb8a03.png)

向下滚动并点击“创建”按钮。

![](img/4405d046100659f04ee740e24350e702.png)

这将为此微服务创建一个路由，以便可以从 OpenShift 容器平台外部调用它。

![](img/22cb3e9621be9ea825c8d8eaa1dfcba1.png)

您可以在上面的“交通”部分看到创建的路线。从 OpenShift 容器平台外部调用服务的 URL 为“[”http://mphelloworld-my project . 192 . 168 . 1 . 6 . xip . io](http://mphelloworld-myproject.192.168.1.6.xip.io/)”。

### 将微服务关联到 JDG

正如你在新的 MicroProfile Java 代码中看到的，更新后的微服务使用了一些环境变量。 为了让微服务“mphelloworld”能够与 JDG 通信，我们需要在其 DeploymentConfig 中包含一些环境变量。第一个变量是微服务将用来调用 JDG 的 REST url。 正如我们之前提到的，REST 调用进入 JDG 的 url 的语法如下:

*< URL > /rest/ <缓存名称> / <缓存关键字>*

在我们的例子中，< URL >将是在前面部分创建的路由，“rest”保持不变，<缓存名称>为“默认”。

为了设置这些环境变量，我们需要对微服务 pod 的部署配置进行更改。首先，通过在笔记本电脑的“终端”窗口输入以下命令，确保您已经登录到 OpenShift 集群:

*$ oc 登录 https://127 . 0 . 0 . 1:8443-u developer-p developer*

![](img/8d86936c5ada875f9d8c10d998e98e60.png)

然后输入以下命令:

*$ app _ path = $(oc describe route datagrid-app | grep " ^requested 主机:" | awk ' { print $ 3 } ')*
*$oc env DC mphelloworld-e URL _ cache _ name =[http://$ { app _ path }/rest/default/](blank)-e num _ of _ data _ items = 5*

![](img/e8d29098bd862bac54d9ffe6b4dfc123.png)

这将导致“mphello world”micro profile pod 的更新，并将由 OpenShift 容器平台自动重启。

### 调用基于微文件的微服务

要测试 MicroProfile 微服务，打开浏览器窗口，输入:

[【http://mphelloworld-myproject.192.168.1.6.xip.io/hello】](http://mphelloworld-myproject.192.168.1.6.xip.io/hello)

你应该看到字符串“你好<名字>来自 MicroProfile 微服务”，这是微服务返回的 JSON，出现在浏览器窗口:

![](img/ba16e7fcc4b73bf317e58c43509d6288.png)

注意，每次通过刷新网页来调用这个 REST 服务时，应该会在问候语中看到一个不同的名称:

![](img/12ea50e144bf0acc7bcf1efbe37b8b11.png)

关于 OpenShift 容器平台上的 Eclipse MicroProfile 的 3 部分系列文章到此结束。我希望你喜欢它，如果想了解更多关于 Eclipse MicroProfile 的信息，请参考下面的链接:

[Eclipse 微档案网站](http://microprofile.io)

[Eclipse MicroProfile Google group](https://groups.google.com/forum/#!forum/microprofile)

[红帽 OpenShift 应用运行时](https://developers.redhat.com/products/rhoar/overview/)

[野生蜂群网站](http://wildfly-swarm.io)

[【野蜂群技术预览版(MicroProfile 1.2 实现)](https://github.com/MicroProfileJWT/wildfly-swarm-mp1.2/releases/tag/MP_12-RC6)

要获得关于 MicroProfile 1.2 技术预览版的帮助，请在 [野生蜂群论坛](https://groups.google.com/forum/#!forum/wildfly-swarm) 输入您的问题

* * *

[**打造你的 Java EE 微服务** **访问**](https://groups.google.com/forum/#!forum/wildfly-swarm) [**野虫群**](https://developers.redhat.com/promotions/wildflyswarm-cheatsheet/) **下载备忘单。**

*Last updated: October 23, 2017*