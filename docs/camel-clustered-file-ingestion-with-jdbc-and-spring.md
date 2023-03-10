# Camel 集群文件摄取与 JDBC 和春天

> 原文：<https://developers.redhat.com/blog/2017/09/08/camel-clustered-file-ingestion-with-jdbc-and-spring>

读取文件是 Apache Camel 的常见用法。从使用一个文件开始一条更长的路线到仅仅需要存储文件内容，只读取一次文件的能力是很重要的。当您有一台部署了路由的服务器时，这很容易，但是当您将路由部署到多台服务器时呢？谢天谢地，Camel 有一个[幂等消费者](http://camel.apache.org/idempotent-consumer.html)的概念。

实现这一点的一个有用方法是使用幂等知识库。这个存储库将跟踪正在读取的文件，并且不允许其他服务器读取它。它就像文件上的锁一样。文件读取完成后，可以选择从数据库中删除该行，这将允许以后接收同名文件，或者保留该行，不允许以后接收同名文件。

那么，如何使用 JDBC 和 Spring 建立一个幂等知识库呢？

首先，您需要创建表和必要的 spring beans。这里我使用的是 JDBCMessageIdRepository。在开发的时候，[这个 bug](https://issues.apache.org/jira/browse/CAMEL-11630) 仍然是一个问题，所以我不能使用 JPA 版本。这个问题已经被解决了，你可以很容易地把一个换成另一个。这个例子假设您已经有了一个用于数据源设置的 bean。您可以对部署中的多个路由使用同一个 fileConsumerRepo bean。

```
 CREATE TABLE CAMEL_MESSAGEPROCESSED ( processorName VARCHAR(255), messageId VARCHAR(100), createdAt TIMESTAMP );
```

```
 @Bean
 public JdbcMessageIdRepository fileConsumerRepo() {
     JdbcMessageIdRepository fileConsumerRepo = null;
     try {
         fileConsumerRepo = new JdbcMessageIdRepository(dataSource(), "fileConsumerRepo");
     } catch (Exception ex) {
         LOGGER.info("############ Caught exception inside Creating fileConsumerRepo ..." + ex.getMessage());
     }
     if (fileConsumerRepo == null) {
         LOGGER.info("############ fileConsumerRepo == null ...");
     }
     return fileConsumerRepo;
 }
```

此外，根据您的具体设置，确保在要扫描的包或持久性单元中包含 org . Apache . camel . processor . idempotent . JPA。在我的例子中，我为 JPA 使用了一个 localcontainereentitymanagerfactorybean。

```
 @Bean
 public LocalContainerEntityManagerFactoryBean entityManagerFactory() throws NamingException {
     final LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();
     em.setPersistenceUnitName("camel");
     final HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
     vendorAdapter.setDatabase(Database.ORACLE);
     vendorAdapter.setShowSql(false);
     vendorAdapter.setGenerateDdl(false);
     em.setJpaVendorAdapter(vendorAdapter);
     em.setPackagesToScan(new String[] { "com.my.package.model","org.apache.camel.processor.idempotent.jpa" });
     em.setJpaProperties(additionalProperties());
     em.setDataSource(this.dataSource());
     return em;
 }
```

现在，您的幂等库已经准备好供骆驼使用了。下面是一个例子

```
from("file:/my/test/dir?moveFailed=.error&autoCreate=true&readLockLoggingLevel=WARN&shuffle=true&readLock=idempotent&idempotentRepository=#fileConsumerRepo&readLockRemoveOnCommit=true")
    .routeId("ingestionFile")
    .convertBodyTo(String.class)
    .log(LoggingLevel.INFO, "File received");
```

让我们来分解一下选项:

| **选项** | **意为** |
| 移动失败 | 如果失败，文件应该放在哪里 |
| 自动创建 | 如果目录不存在，是否自动创建 |
| readlocklogjinglevel | 如果路由无法获得读取文件的锁，应该在哪个级别进行记录 |
| 僵局 | 要使用的读取锁定类型 |
| 幂等存储 | 定义的 spring bean 的名称 |
| readLockRemoveOnCommit | 如果在文件读取完成后应移除该行 |

现在，将您的路线部署到多台服务器上，看看神奇的效果吧！通过使用端口偏移量，您可以使用 EAP 或 Karaf 轻松地做到这一点。如果你正在寻找更多的例子，看看这个由红帽匠乔希·里根合作的项目。

* * *

**无论你是容器新手还是有经验的人，下载这个** [**备忘单**](https://developers.redhat.com/promotions/docker-cheatsheet/) **可以在遇到你最近没有完成的任务时帮助你。**

*Last updated: September 22, 2017*