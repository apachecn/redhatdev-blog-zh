# 对外部数据库使用 Red Hat 的单点登录技术，第 1 部分:使用 MariaDB 安装和配置 SSO

> 原文：<https://developers.redhat.com/blog/2021/05/12/using-red-hats-single-sign-on-technology-with-external-databases-part-1-install-and-configure-sso-with-mariadb>

Red Hat 的[单点登录(SSO)技术](https://access.redhat.com/announcements/3567891)，基于 [Keycloak](https://www.keycloak.org/) 开源项目，是 Red Hat 用于保护 web 应用和 RESTful web 服务的解决方案。Red Hat 单点登录技术的目标是使安全性变得简单，以便应用程序开发人员可以轻松保护他们在组织中部署的应用程序和服务。

开箱即用，单点登录使用自己的基于 Java 的嵌入式关系数据库，称为 H2，来存储持久数据。但是，这种 H2 数据库在高并发情况下是不可行的，不应该在集群中使用。因此，强烈建议用一个更适合生产的外部数据库替换这个 H2 数据库。

本文向您展示了如何安装和配置 Red Hat 的单点登录技术，以连接到更成熟的数据库。我们将使用 MariaDB 作为示例，但是这些说明应该也适用于 MySQL。

**注意**:本文是关于对外部数据库使用 SSO 的系列文章中的第一篇。我的下一篇文章将涉及在 [Red Hat OpenShift](/products/openshift/overview) 中部署 SSO 映像。您将学习如何使用自定义 JDBC 驱动程序将映像连接到运行在 OpenShift 之外的外部 MariaDB 或 MySQL 服务器。

## 设置和配置 MariaDB 或 MySQL 数据库

要执行本节中的步骤，您必须首先以`root`的身份登录到[Red Hat Enterprise Linux](/products/rhel/overview)(RHEL)7 系统。

### 步骤 1:安装和配置 MariaDB 或 MySQL 数据库服务器

这一步可以通过以下命令完成:

```
[root@testvm ~]# yum install mariadb-server mariadb mysql-connector-java

[root@testvm ~]# systemctl enable mariadb

[root@testvm ~]# systemctl start mariadb

[root@testvm ~]# /usr/bin/mysql_secure_installation
...
Set root password? [Y/n] Y
New password: redhat
Re-enter new password: redhat
...
Remove anonymous users? [Y/n] Y
...
Disallow root login remotely? [Y/n] Y
...
Remove test database and access to it? [Y/n] Y
...
Reload privilege tables now? [Y/n] Y
...

All done! If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[root@testvm ~]#

[root@testvm ~]# mysql -h localhost -uroot -predhat
...
MariaDB [(none)]> show databases;
+--------------------+
| Database |
+--------------------+
| information_schema |
| mysql |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)

MariaDB [(none)]>
MariaDB [(none)]> quit
Bye
[root@testvm ~]#
```

### 步骤 2:为 SSO 创建新用户和数据库

输入以下命令创建新用户:

```
[root@testvm ~]# cat mariadb_rhsso74_db_setup.sql
CREATE USER 'rhsso74'@'%' IDENTIFIED BY 'redhat';

DROP DATABASE IF EXISTS `rhsso74`;

CREATE DATABASE IF NOT EXISTS `rhsso74`;

GRANT ALL PRIVILEGES ON rhsso74.* TO 'rhsso74'@'%' identified by 'redhat';

[root@testvm ~]#

[root@testvm ~]# mysql -h localhost -uroot -predhat < mariadb_rhsso74_db_setup.sql
[root@testvm ~]#

[root@testvm ~]# mysql -h localhost -uroot -predhat

MariaDB [(none)]> show databases;
+--------------------+
| Database |
+--------------------+
| *information_schema* |
| mysql |
| performance_schema |
| rhsso74  |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [rhsso74]> quit
Bye
[root@testvm ~]#

[root@testvm ~]# mysql -h localhost -urhsso74 -predhat
MariaDB [(none)]> show databases;
+--------------------+
| Database |
+--------------------+
| *information_schema* |
| rhsso74  |
+--------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> use `rhsso74`;
Database changed
MariaDB [rhsso74]> show tables;
Empty set (0.00 sec)

MariaDB [rhsso74]> quit
Bye
[root@testvm ~]#
```

## 安装和配置 SSO

要执行本节中的步骤，请在 RHEL 7 系统上以`ssoadmin`身份登录。

### 第一步:下载并安装红帽的单点登录 7.4 软件

在撰写本文时， [SSO 7.4 是最新的可用版本](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=core.service.rhsso)。在您选择的目录路径下下载并安装 SSO，例如:

```
/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/

```

创建目录后，运行以下命令安装软件:

```
[ssoadmin@testvm LATEST]$ id -a
uid=1000(ssoadmin) gid=1000(ssoadmin) groups=1000(ssoadmin),10(wheel) context=system_u:system_r:unconfined_service_t:s0
[ssoadmin@testvm LATEST]$

[ssoadmin@testvm LATEST]$ pwd
/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/
[ssoadmin@testvm LATEST]$

[ssoadmin@testvm LATEST]$ unzip rh-sso-7.4.0.zip
Archive: rh-sso-7.4.0.zip
creating: rh-sso-7.4/
creating: rh-sso-7.4/modules/
...
...
inflating: rh-sso-7.4/docs/licenses-rh-sso/org.keycloak,keycloak-server-spi-private,9.0.3.redhat-00002,Apache Software License 2.0.txt
[ssoadmin@testvm LATEST]$

[ssoadmin@testvm LATEST]$ cd rh-sso-7.4/
[ssoadmin@testvm rh-sso-7.4]$

[ssoadmin@testvm rh-sso-7.4]$ cat version.txt
Red Hat Single Sign-On - Version 7.4.0.GA
[ssoadmin@testvm rh-sso-7.4]$
```

### 步骤 2:添加新的管理员用户并启动 SSO 服务器实例

这一步可以通过以下命令完成:

```
[ssoadmin@testvm rh-sso-7.4]$ cd bin
[ssoadmin@testvm bin]$ ./add-user.sh --user admin -p redhat
Updated user 'admin' to file '/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/mgmt-users.properties'
Updated user 'admin' to file '/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/domain/configuration/mgmt-users.properties'
[ssoadmin@testvm bin]$
[ssoadmin@testvm bin]$ ./standalone.sh
```

### 步骤 3:更改 SSO 绑定地址以监听所有网络接口

默认情况下，红帽的单点登录技术通过[红帽 JBoss 企业应用平台](/products/eap/overview) (JBoss EAP)绑定到本地主机环回地址 127.0.0.1。然而，如果您希望认证服务器在您的网络上可用，这不是一个非常有用的默认设置。因此，您需要设置您的网络接口来绑定到除 localhost 之外的东西。您也可以在启动服务器实例时这样做，方法是将`-b`选项设置为您选择的 IP 绑定地址。但是最好只在配置中设置一次，它适用于主机上的所有网络接口。

打开新的终端选项卡，运行以下 JBoss 命令行界面(CLI)命令:

```
[ssoadmin@testvm bin]$ ./jboss-cli.sh --connect
[standalone@localhost:9990 /] /interface=public:write-attribute(name=inet-address,value=0.0.0.0)
[standalone@localhost:9990 /] /interface=management:write-attribute(name=inet-address,value=0.0.0.0)
[standalone@localhost:9990 /] reload
[standalone@localhost:9990 /] exit
```

### 步骤 4:为 SSO 添加新的管理员用户

该用户帐户特定于 SSO 的服务器运行时，并允许您登录到`master`领域的管理控制台，以在 SSO 中执行管理任务。这些任务包括创建领域、创建用户、为应用程序注册客户机以通过单点登录来保护，等等。

添加用户帐户时，命令参数会略有不同，具体取决于您使用的是独立操作模式还是域操作模式。以下内容适用于独立模式:

```
[ssoadmin@testvm bin]$ ./add-user-keycloak.sh -r master -u admin -p redhat
Added 'admin' to '/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/keycloak-add-user.json', restart server to load user
[ssoadmin@testvm bin]$
```

**注意**:如果您的服务器正在运行并且可以从本地主机访问，您也可以通过转到`http://localhost:8080/auth`来创建这个管理员用户。

对于域模式，您必须使用`-sc`选项将脚本指向您的服务器主机之一。例如:

```
[ssoadmin@testvm bin]$ ./add-user-keycloak.sh --sc domain/servers/server-one/configuration -r master -u *username* -p *password*
```

### 步骤 5:重新启动 SSO 服务器实例来加载新的管理员用户

重新启动 SSO 服务器实例:

```
[ssoadmin@testvm bin]$ ./standalone.sh
...
22:20:04,239 INFO [org.jboss.as] (MSC service thread 1-1) WFLYSRV0049: Red Hat Single Sign-On 7.4.0.GA (WildFly Core 10.1.2.Final-redhat-00001) starting
...
...
22:20:22,469 INFO [org.keycloak.services] (ServerService Thread Pool -- 68) KC-SERVICES0006: Importing users from '/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/keycloak-add-user.json'
22:20:23,322 INFO [org.keycloak.services] (ServerService Thread Pool -- 68) KC-SERVICES0009: Added user 'admin' to realm 'master'
...
22:20:23,948 INFO [org.jboss.as] (Controller Boot Thread) WFLYSRV0060: Http management interface listening on http://0.0.0.0:9990/management
22:20:23,948 INFO [org.jboss.as] (Controller Boot Thread) WFLYSRV0051: Admin console listening on http://0.0.0.0:9990
22:20:23,949 INFO [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: Red Hat Single Sign-On 7.4.0.GA (WildFly Core 10.1.2.Final-redhat-00001) started in 19857ms - Started 598 of 893 services (602 services are lazy, passive or on-demand)
...
```

## 配置 SSO 以使用 MariaDB 或 MySQL 数据库

这些步骤与上一节中的步骤一样，在 RHEL 7 系统上作为`ssoadmin`执行。

### 步骤 1:将 MySQL JDBC 驱动程序模块添加到 SSO

为此步骤输入以下命令:

```
[ssoadmin@testvm bin]$ pwd
/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/bin
[ssoadmin@testvm bin]$
[ssoadmin@testvm bin]$ ./jboss-cli.sh
You are disconnected at the moment. Type 'connect' to connect to the server or 'help' for the list of supported commands.
[disconnected /]
[disconnected /] module add --name=com.mysql --resources=/usr/share/java/mysql-connector-java.jar --dependencies=javax.api,javax.transaction.api
[disconnected /] exit

[ssoadmin@testvm bin]$ ls -alrt /home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/modules/com/mysql/main/
total 868
drwxrwxr-x. 3 ssoadmin ssoadmin   18 Oct 17 19:31 ..
-rw-rw-r--. 1 ssoadmin ssoadmin 883899 Oct 17 19:31 mysql-connector-java.jar
drwxrwxr-x. 2 ssoadmin ssoadmin   56 Oct 17 19:31 .
-rw-rw-r--. 1 ssoadmin ssoadmin   317 Oct 17 19:31 module.xml
[ssoadmin@testvm bin]$
```

### 步骤 2:注册数据库驱动程序并添加数据源

注册 MySQL 或 MariaDB 驱动程序:

```
[ssoadmin@testvm bin]$ ./jboss-cli.sh --connect
[standalone@localhost:9990 /]
[standalone@localhost:9990 /] /subsystem=datasources/jdbc-driver=mysql:add(driver-name=mysql,driver-module-name=com.mysql,driver-xa-datasource-class-name=com.mysql.jdbc.jdbc2.optional.MysqlXADataSource, driver-class-name=com.mysql.jdbc.Driver)
{"outcome" => "success"}

[standalone@localhost:9990 /]
[standalone@localhost:9990 /] /subsystem=datasources/data-source=KeycloakMariaDBDS:add(jndi-name=java:jboss/datasources/KeycloakMariaDBDS, driver-name=mysql, connection-url=jdbc:mysql://localhost:3306/rhsso74, user-name=rhsso74, password=redhat)
{"outcome" => "success"}

[standalone@localhost:9990 /]
[standalone@localhost:9990 /] /subsystem=datasources/data-source=KeycloakMariaDBDS:test-connection-in-pool
{
"outcome" => "success",
"result" => [true]
}

[standalone@localhost:9990 /]
[standalone@localhost:9990 /] reload
[standalone@localhost:9990 /]
[standalone@localhost:9990 /] exit
[ssoadmin@testvm bin]$
```

### 步骤 3:更新 SSO JPA 连接以指向新的数据源

在更新 JPA 连接之前，您必须停止 SSO 的服务器实例:

```
[ssoadmin@testvm bin]$ cp -p /home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/standalone.xml /home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/standalone.xml.SAVE_Backup
[ssoadmin@testvm bin]$
[ssoadmin@testvm bin]$ vi /home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/standalone.xml
...
   <subsystem >
     ...
  <spi name="connectionsJpa">
   <provider name="default" enabled="true">
   <properties>
 **<!--** <property name="dataSource" value="java:jboss/datasources/KeycloakDS"/> **-->**
 <property name="dataSource" value="java:jboss/datasources/KeycloakMariaDBDS"/>
       <property name="initializeEmpty" value="true"/>
       <property name="migrationStrategy" value="update"/>
       <property name="migrationExport" value="${jboss.home.dir}/keycloak-database-update.sql"/>
     </properties>
 </provider>
  </spi>
 ...
...
[ssoadmin@testvm bin]$
```

### 步骤 4:重新启动 SSO

在此步骤中，您将重新启动服务器实例，并检查以确保其数据库模型已加载到数据库中:

```
[ssoadmin@testvm bin]$ ./standalone.sh
...
21:14:37,090 INFO [org.jboss.as] (MSC service thread 1-2) WFLYSRV0049: Red Hat Single Sign-On 7.4.0.GA (WildFly Core 10.1.2.Final-redhat-00001) starting
...
21:14:42,034 INFO [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-6) WFLYJCA0001: Bound data source [java:jboss/datasources/KeycloakMariaDBDS]

...

21:14:49,568 INFO [org.keycloak.connections.jpa.updater.liquibase.LiquibaseJpaUpdaterProvider] (ServerService Thread Pool -- 67) **Initializing database schema. Using changelog META-INF/jpa-changelog-master.xml**

...
21:15:02,735 INFO [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: Red Hat Single Sign-On 7.4.0.GA (WildFly Core 10.1.2.Final-redhat-00001) started in 26903ms - Started 598 of 893 services (602 services are lazy, passive or on-demand)
```

连接到 MariaDB 或 MySQL 数据库，并检查 SSO 数据库模型:

```
[root@testvm ~]# mysql -h localhost -urhsso74 -predhat

MariaDB [(none)]> show databases;
+--------------------+
| Database |
+--------------------+
| information_schema |
| rhsso74 |
+--------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]>
MariaDB [(none)]> use `rhsso74`;

MariaDB [rhsso74]> show tables;
+-------------------------------+
| Tables_in_rhsso74 |
+-------------------------------+
| ADMIN_EVENT_ENTITY |
| ASSOCIATED_POLICY |
| AUTHENTICATION_EXECUTION |
| AUTHENTICATION_FLOW |
| AUTHENTICATOR_CONFIG |
| AUTHENTICATOR_CONFIG_ENTRY |
| BROKER_LINK |
| CLIENT |
| CLIENT_ATTRIBUTES |
| CLIENT_AUTH_FLOW_BINDINGS |
| CLIENT_DEFAULT_ROLES |
| CLIENT_INITIAL_ACCESS |
| CLIENT_NODE_REGISTRATIONS |
| CLIENT_SCOPE |
| CLIENT_SCOPE_ATTRIBUTES |
| CLIENT_SCOPE_CLIENT |
| CLIENT_SCOPE_ROLE_MAPPING |
| CLIENT_SESSION |
| CLIENT_SESSION_AUTH_STATUS |
| CLIENT_SESSION_NOTE |
| CLIENT_SESSION_PROT_MAPPER |
| CLIENT_SESSION_ROLE |
| CLIENT_USER_SESSION_NOTE |
| COMPONENT |
| COMPONENT_CONFIG |
| COMPOSITE_ROLE |
| CREDENTIAL |
| DATABASECHANGELOG |
| DATABASECHANGELOGLOCK |
| DEFAULT_CLIENT_SCOPE |
| EVENT_ENTITY |
| FEDERATED_IDENTITY |
| FEDERATED_USER |
| FED_USER_ATTRIBUTE |
| FED_USER_CONSENT |
| FED_USER_CONSENT_CL_SCOPE |
| FED_USER_CREDENTIAL |
| FED_USER_GROUP_MEMBERSHIP |
| FED_USER_REQUIRED_ACTION |
| FED_USER_ROLE_MAPPING |
| GROUP_ATTRIBUTE |
| GROUP_ROLE_MAPPING |
| IDENTITY_PROVIDER |
| IDENTITY_PROVIDER_CONFIG |
| IDENTITY_PROVIDER_MAPPER |
| IDP_MAPPER_CONFIG |
| KEYCLOAK_GROUP |
| KEYCLOAK_ROLE |
| MIGRATION_MODEL |
| OFFLINE_CLIENT_SESSION |
| OFFLINE_USER_SESSION |
| POLICY_CONFIG |
| PROTOCOL_MAPPER |
| PROTOCOL_MAPPER_CONFIG |
| REALM |
| REALM_ATTRIBUTE |
| REALM_DEFAULT_GROUPS |
| REALM_DEFAULT_ROLES |
| REALM_ENABLED_EVENT_TYPES |
| REALM_EVENTS_LISTENERS |
| REALM_REQUIRED_CREDENTIAL |
| REALM_SMTP_CONFIG |
| REALM_SUPPORTED_LOCALES |
| REDIRECT_URIS |
| REQUIRED_ACTION_CONFIG |
| REQUIRED_ACTION_PROVIDER |
| RESOURCE_ATTRIBUTE |
| RESOURCE_POLICY |
| RESOURCE_SCOPE |
| RESOURCE_SERVER |
| RESOURCE_SERVER_PERM_TICKET |
| RESOURCE_SERVER_POLICY |
| RESOURCE_SERVER_RESOURCE |
| RESOURCE_SERVER_SCOPE |
| RESOURCE_URIS |
| ROLE_ATTRIBUTE |
| SCOPE_MAPPING |
| SCOPE_POLICY |
| USERNAME_LOGIN_FAILURE |
| USER_ATTRIBUTE |
| USER_CONSENT |
| USER_CONSENT_CLIENT_SCOPE |
| USER_ENTITY |
| USER_FEDERATION_CONFIG |
| USER_FEDERATION_MAPPER |
| USER_FEDERATION_MAPPER_CONFIG |
| USER_FEDERATION_PROVIDER |
| USER_GROUP_MEMBERSHIP |
| USER_REQUIRED_ACTION |
| USER_ROLE_MAPPING |
| USER_SESSION |
| USER_SESSION_NOTE |
| WEB_ORIGINS |
+-------------------------------+
93 rows in set (0.01 sec)

MariaDB [rhsso74]> select ID, NAME, ACCOUNT_THEME, ADMIN_THEME, DEFAULT_LOCALE from REALM;
+----------------+----------------+------------------+-------------+----------------+
| ID             | NAME           | ACCOUNT_THEME    | ADMIN_THEME | DEFAULT_LOCALE |
+----------------+----------------+------------------+-------------+----------------+
| master         | master         | keycloak-preview | NULL        | NULL           |
+----------------+----------------+------------------+-------------+----------------+
1 rows in set (0.00 sec)

MariaDB [rhsso74]>
```

### 步骤 5:为 SSO 重新创建管理员用户

这一步是必需的，因为您已经将 SSO 正在使用的数据库切换到 MariaDB 或 MySQL。前面创建的 admin 用户位于默认的 H2 数据库中，该数据库不再使用:

```
[ssoadmin@testvm bin]$ ./add-user-keycloak.sh -r master -u admin -p redhat
Added 'admin' to '/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/keycloak-add-user.json', restart server to load user
[ssoadmin@testvm bin]$

```

### 步骤 6:最终测试和验证

启动 SSO 服务器实例:

```
[ssoadmin@testvm bin]$ ./standalone.sh
...
22:20:04,239 INFO [org.jboss.as] (MSC service thread 1-1) WFLYSRV0049: Red Hat Single Sign-On 7.4.0.GA (WildFly Core 10.1.2.Final-redhat-00001) starting
...
22:20:22,469 INFO [org.keycloak.services] (ServerService Thread Pool -- 68) KC-SERVICES0006: Importing users from '/home/ssoadmin/RH-SSO_SETUP/RH-SSO_7.x/7.4.x/LATEST/rh-sso-7.4/standalone/configuration/keycloak-add-user.json'
22:20:23,322 INFO [org.keycloak.services] (ServerService Thread Pool -- 68) KC-SERVICES0009: Added user 'admin' to realm 'master'

...

22:20:23,949 INFO [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: Red Hat Single Sign-On 7.4.0.GA (WildFly Core 10.1.2.Final-redhat-00001) started in 19857ms - Started 598 of 893 services (602 services are lazy, passive or on-demand)
```

验证已在数据库中创建了管理员用户:

```
MariaDB [rhsso74]> select ID, USERNAME, REALM_ID from USER_ENTITY;
+--------------------------------------+------------------+----------------+
| ID | USERNAME | REALM_ID       |
+--------------------------------------+------------------+----------------+
| 075d1cb9-fb4d-4437-80f8-f4397b1b5370 | admin       | master         |
+--------------------------------------+------------------+----------------+
1 row in set (0.00 sec)

MariaDB [rhsso74]>
```

通过打开浏览器、加载`http://localhost:8080/auth`，并以`admin/redhat`的身份登录，访问 SSO 服务器实例中的管理控制台。

## SSO 的后续步骤

为了继续学习使用红帽的单点登录技术，你可以在 GitHub 上玩玩 [SSO 快速入门](https://github.com/redhat-developer/redhat-sso-quickstarts)和 [Keycloak 快速入门](https://github.com/keycloak/keycloak-quickstarts)。另外，查看一下[红帽的单点登录技术](https://access.redhat.com/documentation/en/red-hat-single-sign-on/)和 [SSO 7.4 版本](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.4/html/release_notes/index)的文档。

## 接下来...

本系列的下一篇文章将向您展示如何在 OpenShift 中部署 SSO 容器映像。未来的文章将涵盖一些主题和用例，比如跨不同数据库风格的 SSO 迁移和升级，以及使用自定义 JDBC 驱动程序连接到外部数据库的技巧和提示。我还将分享 Red Hat 单点登录技术的更强大的功能和其他与 SSO 集成相关的主题，如与第三方 OpenID Connect 和安全断言标记语言(SAML)供应商的集成，在使用 Nginx 作为负载平衡器或反向代理的同时进行集群，以及在 SSO 中使用代码交换(PKCE)扩展的证明密钥。敬请期待！

*Last updated: October 14, 2022*