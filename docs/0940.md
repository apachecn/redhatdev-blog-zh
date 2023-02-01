# 如何用 Karaf 为 Red Hat Fuse 7 配置 JDBC 附加器

> 原文：<https://developers.redhat.com/blog/2019/01/23/how-to-configure-jdbc-appender-for-red-hat-fuse-7-wiht-karaf>

对于一些集成项目来说，将应用程序日志保存在关系数据库而不是普通的日志文件中是很有帮助的。在本文中，我将展示如何为运行在 Apache Karaf 环境中的 [Red Hat Fuse 7](https://developers.redhat.com/products/fuse/overview/) 配置一个 JDBC 附加器。有说明描述了[如何用 PostgreSQL](https://issues.jboss.org/browse/FUSEDOC-2969) 持久化消息。相反，我将展示如何为 Oracle Database 11g 设置一个 JDBC 附加器。

我已经用 Oracle Database 11g Express Edition 测试了这个过程。我发现的一个主要区别是表语法，以及需要 Oracle Database 11g 序列和触发器来自动生成主键。因此，Oracle Database 11g 的用户应该会发现本文非常有用。

1.在`FUSE_HOME/etc`文件夹中创建一个`org.ops4j.datasource-oracleds.cfg`文件，内容如下:

```
databaseName=xe
user=SYSTEM
password=administrator
dataSourceName=oracleds
url=jdbc:oracle:thin:@192.168.1.10:1521/xe
osgi.jdbc.driver.class=oracle.jdbc.OracleDriver
```

2.使用以下命令从 Karaf 终端安装 Oracle 驱动程序:

```
 osgi:install -s wrap:mvn:com.oracle/ojdbc6/11.2.0.3

```

3.安装这些功能:

```
feature:install jdbc jndi pax-jdbc-oracle

```

4.检查`DataSourceFactory`对象的`service:list`:

```
karaf@root()> service:list DataSourceFactory
[org.osgi.service.jdbc.DataSourceFactory]
-----------------------------------------
 osgi.jdbc.driver.class = oracle.jdbc.OracleDriver
 osgi.jdbc.driver.name = wrap_mvn_com.oracle_ojdbc6_11.2.0.3
 osgi.jdbc.driver.version = 0.0.0
 service.bundleid = 223
 service.id = 259
 service.scope = singleton
Provided by : 
 wrap_mvn_com.oracle_ojdbc6_11.2.0.3 (223)
Used by: 
 OPS4J Pax JDBC Config (239)

[org.osgi.service.jdbc.DataSourceFactory]
-----------------------------------------
 osgi.jdbc.driver.class = oracle.jdbc.OracleDriver
 osgi.jdbc.driver.name = oracle
 service.bundleid = 237
 service.id = 246
 service.scope = singleton
Provided by : 
 OPS4J Pax JDBC Oracle Driver Adapter (237)
Used by: 
 OPS4J Pax JDBC Config (239)

```

5.检查`DataSource`对象的`service:list`:

```
 karaf@root()> service:list DataSource
[javax.sql.DataSource]
----------------------
 databaseName = xe
 dataSourceName = oracleds
 felix.fileinstall.filename = file:/home/cpandey/NotBackedUp/Development/RedHat_Fuse_Folder/fuse7_2/fuse-karaf-7.2.0.fuse-720035-redhat-00001/etc/org.ops4j.datasource-oracleds.cfg
 osgi.jdbc.driver.class = oracle.jdbc.OracleDriver
 osgi.jndi.service.name = oracleds
 password = administrator
 pax.jdbc.managed = true
 service.bundleid = 239
 service.factoryPid = org.ops4j.datasource
 service.id = 261
 service.pid = org.ops4j.datasource.a6fadb91-16e5-43bb-8990-bbbaa553be8d
 service.scope = singleton
 url = jdbc:oracle:thin:@192.168.1.10:1521/xe
 user = SYSTEM
Provided by : 
 OPS4J Pax JDBC Config (239)
Used by: 
 OPS4J Pax Logging - Log4j v2 (8)

```

6.创建事件表:

```
CREATE TABLE "SYSTEM"."EVENTS" 
   (	"EVENT_ID" NUMBER(10,0) NOT NULL ENABLE, 
	"EVENT_DATE" TIMESTAMP (0), 
	"EVENT_LEVEL" VARCHAR2(5), 
	"EVENT_SOURCE" VARCHAR2(128), 
	"EVENT_THREAD_ID" VARCHAR2(128), 
	"EVENT_MESSAGE" VARCHAR2(1024), 
	 CONSTRAINT "EVENT_ID_PK" PRIMARY KEY ("EVENT_ID"));

```

table create 语句也可以从 Karaf 终端执行:

```
karaf@root()> jdbc:execute oracleds 'CREATE TABLE "EVENTS" ("EVENT_ID" NUMBER(10,0) NOT NULL ENABLE, "EVENT_DATE" TIMESTAMP (0), "EVENT_LEVEL" VARCHAR2(5), "EVENT_SOURCE" VARCHAR2(128), "EVENT_THREAD_ID" VARCHAR2(128), "EVENT_MESSAGE" VARCHAR2(1024), CONSTRAINT "EVENT_ID_PK" PRIMARY KEY ("EVENT_ID"))'

```

我发现那样做很有用。它还将测试数据源的设置是否正确。

7.在数据库中，我们需要创建一个序列和触发器。

序列:

```
create sequence events_seq start with 1 increment by 1;

```

触发器:

```
create or replace trigger events_seq_tr
 before insert on events for each row
 when (new.event_id is null)
begin
 select events_seq.nextval into :new.event_id from dual;
END;

```

请注意触发器中的终止分号。

8.在`FUSE_HOME/etc/org.ops4j.pax.logging.cfg`中，设置以下内容:

```
log4j2.rootLogger.appenderRef.JdbcAppender.ref = JdbcAppender

log4j2.appender.jdbc.type = JDBC
log4j2.appender.jdbc.name = JdbcAppender
log4j2.appender.jdbc.tableName = EVENTS
log4j2.appender.jdbc.cs.type = DataSource
log4j2.appender.jdbc.cs.jndiName = osgi:service/oracleds
log4j2.appender.jdbc.c1.type = Column
log4j2.appender.jdbc.c1.name = EVENT_DATE
log4j2.appender.jdbc.c1.isEventTimestamp = true
log4j2.appender.jdbc.c2.type = Column
log4j2.appender.jdbc.c2.name = EVENT_LEVEL
log4j2.appender.jdbc.c2.pattern = %level
# setNString vs setString
log4j2.appender.jdbc.c2.isUnicode = false
log4j2.appender.jdbc.c3.type = Column
log4j2.appender.jdbc.c3.name = EVENT_SOURCE
log4j2.appender.jdbc.c3.pattern = %logger
log4j2.appender.jdbc.c3.isUnicode = false
log4j2.appender.jdbc.c4.type = Column
log4j2.appender.jdbc.c4.name = EVENT_THREAD_ID
log4j2.appender.jdbc.c4.pattern = %thread
log4j2.appender.jdbc.c4.isUnicode = false
log4j2.appender.jdbc.c5.type = Column
log4j2.appender.jdbc.c5.name = EVENT_MESSAGE
log4j2.appender.jdbc.c5.pattern = %message
log4j2.appender.jdbc.c5.isUnicode = false

```

9.使用以下命令进行测试:

```
karaf@root()> log:log "hey test 123"

```

10.您将得到的 Red Hat Fuse 日志:

```
2019-01-04 19:26:46,098 | INFO  | pipe-log:set INFO    | o.a.k.l.core                     | 135 - org.apache.karaf.log.core - 4.2.0.fuse-720061-redhat-00001 | hey csp 123
2019-01-04 19:29:18,273 | INFO  | g:log "hey test 123" | o.a.k.l.core                     | 135 - org.apache.karaf.log.core - 4.2.0.fuse-720061-redhat-00001 | hey test 123

```

11.在数据库中，执行`select * from events;`:

```
EVENT_ID EVENT_DATE           EVENT_LEVEL   EVENT_SOURCE                 EVENT_THREAD_ID        EVENT_MESSAGE 
199	2019-01-04 19:25:42	INFO	org.apache.karaf.log.core	pipe-log:set INFO	hey csp
200	2019-01-04 19:26:08	INFO	org.apache.karaf.log.core	pipe-log:set INFO	hey csp123
201	2019-01-04 19:26:46	INFO	org.apache.karaf.log.core	pipe-log:set INFO	hey csp 123
202	2019-01-04 19:29:18	INFO	org.apache.karaf.log.core	pipe-log:log "hey test 123"	hey test 123

```

就是这样。我希望本文不仅能帮助您在 Oracle Database 11g 上配置用于持久化日志的 JDBC 附加器，还能在 Karaf 环境中使用 Red Hat Fuse 7 配置数据源。它还可以帮助您获得一些使用 Karaf 命令的实践经验。