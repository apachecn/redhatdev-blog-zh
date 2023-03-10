# 带有 JCliff 的 Ansible 集合的 WildFly 服务器配置，第 2 部分

> 原文：<https://developers.redhat.com/blog/2020/12/03/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-2>

欢迎阅读本系列的第二部分，介绍 JCliff 的 [Ansible 集合。这个新的扩展旨在使用 Ansible 微调 WildFly 或](https://github.com/wildfly-extras/ansible_collections_jcliff/releases/tag/v0.0.2)[Red Hat JBoss Enterprise Application Platform](https://developers.redhat.com/products/eap/download)(JBoss EAP)配置。在[第 1 部分](https://developers.redhat.com/blog/2020/11/06/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-1)中，我们安装了 JCliff 及其 Ansible 集合，并准备好了我们的环境。我们为在目标系统上安装 JCliff 建立了一个最小的、可工作的剧本。在本文中，我们将重点关注配置我们的 WildFly 服务器的几个子系统。

## 配置新的数据源

Java 应用程序经常需要使用来自数据库的数据。为了连接到数据库，WildFly 使用了一个驱动程序。WildFly 上托管的多个应用程序甚至可以同时访问一个数据源。在本节中，您将学习如何使用 Ansible 配置数据源子系统。

WildFly 包括一个名为 [H2](https://www.h2database.com/) 的嵌入式内存数据库。在 H2 数据库中，所需的 Java 数据库连接(Java Database Connectivity，JDBC)驱动程序已经准备就绪。以下数据源配置示例已在服务器配置中定义:

```
<subsystem >
            <datasources>
                <datasource jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" use-java-context="true" statistics-enabled="${wildfly.datasources.statistics-enabled:${wildfly.statistics-enabled:false}}">
                    <connection-url>jdbc:h2:mem:test;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE</connection-url>
                    <driver>h2</driver>
                    <security>
                        <user-name>sa</user-name>
                        <password>sa</password>
                    </security>
                </datasource>
...
```

现在，假设我们需要另一个 H2 数据库实例来进行测试。为了设置这个新实例，我们将使用`jcliff:`模块中的`datasources:`元素:

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
    - jcliff:
        wfly_home: "{{ jboss_home }}"
        subsystems:
          - system_props:
              - name: jcliff.enabled
                value: 'enabled.plus'
          - datasources:
              - name: H2DS4Test
                use_java_context: 'true'
                jndi_name: java:jboss/datasources/H2DS4Test
                connection_url: "jdbc:h2:mem:test2;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE"
                driver_name: h2

```

这里，配置也很简单。我们只需添加所需的连接信息。当我们运行剧本时，会添加一个新的数据源。我们可以通过检查 WildFly 的日志文件来验证这一点:

```
...
09:53:03,542 INFO  [org.jboss.as.connector.subsystems.datasources] (MSC service thread 1-6) WFLYJCA0001: Bound data source [java:jboss/datasources/H2DS4Test]
...
```

## 设置新的 JDBC 驱动程序

现在，我们可以使用 Ansible 向 WildFly 添加一个新的 JDBC 驱动程序。首先，我们需要下载驱动程序并将其部署在 JBoss 模块中。我们通过在`${JBOSS_HOME}/modules`目录下创建所需的目录结构并向其中添加 XML 描述符来设置部署。以下示例显示了如何使用 Ansible 将 JDBC 驱动程序添加到 WildFly。

```
...

vars:
  jboss_home: "{{ lookup('env','JBOSS_HOME') }}"
  psql_module_home: "{{ jboss_home }}//modules/org/postgresql/main/"
  jdbc_driver_version: 9.2-1002-jdbc4
  jdbc_driver_jar_filename: "postgresql-{{ jdbc_driver_version }}.jar"
collections:
  - wildfly.jcliff
roles:
  - jcliff

tasks:

  - name: "Set up module dir for Postgres JDBC Driver"
    file:
      path: "{{ psql_module_home }}"
      state: directory
      recurse: yes

  - name: "Ensures WildFly Postgres Driver is present"
    uri:
      url: "https://repo.maven.apache.org/maven2/org/postgresql/postgresql/{{ jdbc_driver_version }}/{{ jdbc_driver_jar_filename }}"
      dest: "{{ psql_module_home }}"
      creates: "{{ psql_module_home }}"

  - name: "Deploy module.xml for Postgres JDBC Driver"
    template:
      src: templates/pgsql_jdbc_driver_module.xml.j2
      dest: "{{ psql_module_home }}/module.xml"

```

通过三项任务，我们已经成功地为 WildFly 安装了一个新的 JDBC 驱动程序。我们仍然需要更新服务器的配置，以使新的驱动程序可用于我们托管的应用程序。我们一会儿就去做。首先，让我们运行行动手册的这一部分，以确保一切按预期运行:

```
$ ansible-playbook playbook.yml
...
TASK [wildfly.jcliff.jcliff : Test if package jcliff is already installed] *******************************************************************************************************************
ok: [localhost]

TASK [wildfly.jcliff.jcliff : Install Jcliff using standalone binary] ************************************************************************************************************************
skipping: [localhost]

TASK [Set up module dir for Postgres JDBC Driver] ********************************************************************************************************************************************
changed: [localhost]

TASK [Ensures WildFly Postgres Driver is present] ********************************************************************************************************************************************
ok: [localhost]

TASK [Deploy module.xml for Postgres JDBC Driver] ********************************************************************************************************************************************
changed: [localhost]

TASK [jcliff] ********************************************************************************************************************************************************************************
ok: [localhost]

PLAY RECAP ***********************************************************************************************************************************************************************************
localhost                  : ok=10   changed=2    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0

```

我们为安装驱动程序所做的一切都使用了 Ansible 现有的原语。现在新的部分来了。多亏了 JCliff，我们可以在服务器中激活驱动程序:

```
   - jcliff:
       wfly_home: "{{ jboss_home }}"
       subsystems:
         - system_props:
             - name: jcliff.enabled
               value: 'enabled.plus'
         - drivers:
             - driver_name: postgresql
               driver_module_name: org.postgresql
               driver_class_name: org.postgresql.Driver
               driver_xa_datasource_class_name: org.postgresql.xa.PGXADataSource  
         - datasources:
             - name: H2DS4Test
               use_java_context: 'true'
               jndi_name: java:jboss/datasources/H2DS4Test
               connection_url: "jdbc:h2:mem:test2;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE"
               driver_name: h2

```

运行剧本后，我们可以看到 WildFly 已经正确地部署了驱动程序。检查服务器的日志文件可以验证这一点:

```
...
13:24:25,474 INFO  [org.jboss.as.connector.subsystems.datasources] (management-handler-thread - 1) WFLYJCA0005: Deploying non-JDBC-compliant driver class org.postgresql.Driver (version 9.2)
13:24:25,474 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-1) WFLYJCA0018: Started Driver service with driver-name = postgresql
13:24:27,464 INFO  [org.jboss.as.connector.deployers.jdbc] (MSC service thread 1-2) WFLYJCA0019: Stopped Driver service with driver-name = postgresql
...
```

## 用 JCliff 部署应用程序

到目前为止，我们只使用 JCliff 来更新 WildFly 的配置。但是它可以走得更远:我们还可以使用 JCliff 在 WildFly 上部署应用程序。在本节中，我们将在 WildFly 服务器上部署 [Jenkins CI](https://www.jenkins.io/) (webapp)。

首先，我们需要添加一个`yum`存储库，这样我们就可以在目标系统上安装所需的文件。在调用`jcliff:`模块之前，将以下内容添加到 Ansible 剧本中:

```
...
   - name: Jenkins Yum Repository
     yum_repository:
         name: jenkins
         description: Jenkins
         baseurl: http://pkg.jenkins.io/redhat
         gpgcheck: 1

   - name: Install Jenkins
     yum:
         name: jenkins
         state: present
...
```

**注意**:这个 Yum 存储库要求你导入一个 Gnu 隐私保护(GPG)密钥，所以在运行剧本之前一定要这样做。输入以下内容以导入密钥: `rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key`。

当剧本成功运行时，将安装与 Jenkins 关联的 RPM 包。我们用来部署应用程序的 WAR 文件将在目标系统上可用:

```
$ yum whatprovides */jenkins*.war
… 

jenkins-2.253-1.1.noarch : Jenkins Automation Server
Repo        : @jenkins
Matched from:
Filename    : /usr/lib/jenkins/jenkins.war

```

现在，我们只需要在 WildFly 服务器上部署应用程序。为此，我们将更新`jcliff:`模块配置以添加`deployments:`元素:

```
   - jcliff:
       wfly_home: "{{ jboss_home }}"
       subsystems:
         - system_props:
             - name: jcliff.enabled
               value: 'enabled.plus'
         - drivers:
             - driver_name: postgresql
               driver_module_name: org.postgresql
               driver_class_name: org.postgresql.Driver
               driver_xa_datasource_class_name: org.postgresql.xa.PGXADataSource  
         - datasources:
             - name: H2DS4Test
               use_java_context: 'true'
               jndi_name: java:jboss/datasources/H2DS4Test
               connection_url: "jdbc:h2:mem:test2;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE"
               driver_name: h2
         - deployments:  
             - name: jenkins
               path: /usr/lib/jenkins/jenkins.war

```

同样，我们只需要添加 WAR 文件的路径和要部署的应用程序的名称。Ansible 和 JCliff 处理剩下的。

剧本成功运行，并且很容易验证 Jenkins 已经被部署。这里，再次检查服务器的日志文件来确认这一点:

```
...
14:06:09,148 INFO  [org.jboss.as.repository] (management-handler-thread - 2) WFLYDR0001: Content added at location /opt/wildfly-20.0.1.Final/standalone/data/content/6b/30899e9a28a361241d19561c965977aea70d7f/content
14:06:09,171 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0027: Starting deployment of "jenkins" (runtime-name: "jenkins")
14:06:09,840 WARN  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0274: Excluded dependency com.fasterxml.jackson.core.jackson-core via jboss-deployment-structure.xml does not exist.
14:06:09,841 WARN  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0274: Excluded dependency org.jboss.resteasy.resteasy-jackson-provider via jboss-deployment-structure.xml does not exist.
14:06:09,841 WARN  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0274: Excluded dependency com.fasterxml.jackson.core.jackson-databind via jboss-deployment-structure.xml does not exist.
14:06:09,841 WARN  [org.jboss.as.server.deployment] (MSC service thread 1-5) WFLYSRV0274: Excluded dependency com.fasterxml.jackson.core.jackson-annotations via jboss-deployment-structure.xml does not exist.
14:06:10,189 INFO  [org.infinispan.PERSISTENCE] (MSC service thread 1-7) ISPN000556: Starting user marshaller 'org.wildfly.clustering.infinispan.marshalling.jboss.JBossMarshaller'
14:06:10,209 INFO  [org.infinispan.CONTAINER] (MSC service thread 1-7) ISPN000128: Infinispan version: Infinispan 'Turia' 10.1.8.Final
14:06:10,501 INFO  [org.jboss.as.clustering.infinispan] (ServerService Thread Pool -- 78) WFLYCLINF0002: Started client-mappings cache from ejb container
14:06:10,557 INFO  [org.jboss.as.server] (management-handler-thread - 2) WFLYSRV0010: Deployed "jenkins" (runtime-name : "jenkins")
...

```

## 结论

第 2 部分到此为止！您已经学习了在行动手册中使用`jcliff:`模块的基本知识，但是仍然有一些高级用例需要探索。我们将在本系列的最后一集[第 3 部分](https://developers.redhat.com/blog/2020/12/21/wildfly-server-configuration-with-ansible-collection-for-jcliff-part-3/)中讨论这个问题。

## 承认

特别感谢 [Andrew Block](https://developers.redhat.com/blog/author/ablock/) 回顾这一系列文章。

*Last updated: December 23, 2020*