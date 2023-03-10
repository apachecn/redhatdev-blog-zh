# 自定义冬眠 ORM 和野生蜂群冬眠搜索

> 原文：<https://developers.redhat.com/blog/2018/03/01/wildfly-swarm-custom-hibernate-versions>

本文将展示如何在一个 [WildFly Swarm](http://wildfly-swarm.io) Java 应用程序上使用自定义版本的 [Hibernate](http://hibernate.org) (Hibernate ORM 和 Hibernate Search)。我不会给出关于野生蜂群配置的细节，如果你需要更多信息你可以看看[野生蜂群用户指南。](https://wildfly-swarm.gitbooks.io/wildfly-swarm-users-guide/)

野生蜂群包括[冬眠群](http://hibernate.org/orm/)和[冬眠搜索。](http://hibernate.org/search/)
你可以用以下方式将分数相加:

```
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>jpa</artifactId>
</dependency>
```

```
<dependency>
 <groupId>org.wildfly.swarm</groupId>
 <artifactId>hibernate-search</artifactId>
</dependency>
```

不幸的是，WildFly Swarm 没有使用最新版本的 [Hibernate ORM](http://hibernate.org/orm/) 和 [Hibernate Search，](http://hibernate.org/search/)，所以你将无法使用 Hibernate 的一些功能，这些功能只有最新版本才有。例如，最新版本的 Hibernate Search 提供了整合[弹性搜索的能力。](https://www.elastic.co/products/elasticsearch)更多信息请访问[休眠搜索文档](https://docs.jboss.org/hibernate/search/5.8/reference/en-US/html_single/#elasticsearch-integration)。

## 解决办法

*   创建一个野生蜂群应用程序。你可以使用野蜂群生成器， [JBoss Forge，](https://forge.jboss.org/addon/org.jboss.forge.addon:wildfly-swarm)或者选择另一个选项。
*   在你的项目中加入野生蜂群(Hibernate Search 和 ORM ):

```
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>datasources</artifactId>
</dependency>
<dependency>
    <groupId>org.wildfly.swarm</groupId>
    <artifactId>hibernate-search</artifactId>
</dependency>
```

*   将 [maven-dependency-plugin](https://maven.apache.org/plugins/maven-dependency-plugin/) 添加到 pom.xml:

```
<plugin>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>unpack</id>
            <phase>generate-resources</phase>
            <goals>
                <goal>unpack</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>org.hibernate</groupId>
                        <artifactId>hibernate-orm-modules</artifactId>
                        <version>5.2.12.Final</version>
                        <classifier>wildfly-11-dist</classifier>
                        <type>zip</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.outputDirectory}/modules</outputDirectory>
                    </artifactItem>
                    <artifactItem>
                        <groupId>org.hibernate</groupId>
                        <artifactId>hibernate-search-modules</artifactId>
                        <version>5.8.2.Final</version>
                        <classifier>wildfly-11-dist</classifier>
                        <type>zip</type>
                        <overWrite>false</overWrite>
                        <outputDirectory>${project.build.outputDirectory}/modules</outputDirectory>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这将把 Hibernate 模块添加到项目的文件夹 **/modules** 中(WildFly Swarm 将检测/modules 文件夹下的所有模块)。关于如何使用 Hibernate 模块的更多信息，请看一下[在 WildFly](https://docs.jboss.org/hibernate/orm/5.2/topical/html_single/wildfly/Wildfly.html) 的 Hibernate ORM 和[在 WildFly](https://docs.jboss.org/hibernate/stable/search/reference/en-US/html_single/#search-configuration-deploy-on-wildfly) 的 Hibernate Search

*   将 persistence.xml 添加到项目中，然后添加 Hibernate 属性:

> ```
> <property name="jboss.as.jpa.providerModule" value="org.hibernate:5.2.12.Final"/>
> 
> <property name="wildfly.jpa.hibernate.search.module" value="org.hibernate.search.orm:5.8.2.Final"/>
> ```

这就是全部，现在你在你的 WildFly Swarm 应用程序上有了你自己的 Hibernate 版本。如果想看代码，这里有一个回购完整的例子:**[https://github.com/carlosthe19916/wildfly-swarm-hibernate](https://github.com/carlosthe19916/wildfly-swarm-hibernate)**

## 警告:

该示例可以在除 2017.12.0 之外的任何版本的 WildFly Swarm 应用程序上运行，因为最近在 2018.1.0 上解决了一些错误。

#### 如果你对这篇文章有任何问题，请告诉我！！

*Last updated: August 29, 2022*