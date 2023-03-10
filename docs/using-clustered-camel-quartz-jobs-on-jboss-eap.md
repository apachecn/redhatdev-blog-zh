# 在 JBoss EAP 上使用集群 Camel Quartz 作业

> 原文：<https://developers.redhat.com/blog/2017/08/10/using-clustered-camel-quartz-jobs-on-jboss-eap>

对于需要在每天特定时间运行的作业，Camel Quartz 可能是一个有用的组件。最近在一个客户网站上，我们需要大约 15 个不同的作业，每个作业都创建不同格式的文件，并将每个文件发送到特定的目的地。虽然在单台机器上进行设置很简单，但是一旦我们开始将 camel routes 部署到多台服务器上，工作就开始在两台机器上进行。为了解决这个问题，我们需要创建一个作业存储。

# 第一步

首先，创建您的 camel routes 并将 camel-quartz 依赖项添加到您的项目中。确保在终端上使用 job.name 选项。这将使作业存储中的存储位置更加清晰。为了使用 oracle 数据库，我们需要以下依赖项。

```
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-quartz2</artifactId>
</dependency>
```

一个示例路由如下所示:

```
from("quartz2://mySampleCronJob?cron=0+0+16+*+*+?&trigger.timeZone=America/Chicago&job.name=myCronJob")
    .log(LoggingLevel.INFO, "Cron job kicked off");
```

# 第二步

创建这个作业存储的下一步是创建表。除了一些索引之外，您还需要创建 *qrtz_job_details、qrtz_triggers、qrtz_simple_triggers、qrtz_cron_triggers、qrtz_simprop_triggers、qrtz_calendars、qrtz _ paused _ triggers _ grps _ qrtz _ fired _ triggers、qrtz_scheduler_state、*和 *qrtz_locks* 。您特定数据库的脚本可以在 quartz-2.2.3/docs/dbTables 文件夹中的 http://www.quartz-scheduler.org/downloads/[这里找到。](http://www.quartz-scheduler.org/downloads/)

# 第三步

最后，您需要配置您的 quartz.properties。如果您计划只在一个环境中，请随意使用名为 *quartz.properties* 的属性文件。然而，在我们的例子中，我们需要考虑多个环境，并且在每个环境中都有不同的数据库。使用 spring，您可以为您的 quartz camel 组件配置一个 bean，它具有所需的所有属性，如下所示。请注意， *SPRING_ACTIVE_PROFILE* 是一个可以在 EAP 实例上设置的系统属性。从那里，代码将查看相应的属性文件，如 *uat.properties.* 该文件应该包含数据库配置值 *db.url* 、 *db.user* 和 *db.password* 。

```
@Configuration
@PropertySource(value={"classpath:${SPRING_ACTIVE_PROFILE}.properties"})
public class SampleConfig{

    @Value("${db.url}")
    private String dbUrl;

    @Value("${db.user}")
    private String dbUser;

    @Value("${db.password}")
    private String dbPassword;

    public SampleConfig(){}

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
         return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean
    public QuartzComponent quartz2(){
        QuartzComponent qc = new QuartzComponent();
        Properties p = new Properties();
        p.put("org.quartz.scheduler.instanceName", "myScheduler");
        p.put("org.quartz.scheduler.instanceId", "AUTO");
        p.put("org.quartz.threadPool.class", "org.quartz.simpl.SimpleThreadPool");
        p.put("org.quartz.threadPool.threadCount", "25");
        p.put("org.quartz.threadPool.threadPriority", "5");
        p.put("org.quartz.jobStore.misfireThreshold", "60000");
        p.put("org.quartz.jobStore.class", "org.quartz.impl.jdbcjobstore.JobStoreTX");
        p.put("org.quartz.jobStore.driverDelegateClass", "org.quartz.impl.jdbcjobstore.oracle.OracleDelegate");
        p.put("org.quartz.jobStore.useProperties", "false");
        p.put("org.quartz.jobStore.dataSource", "myDS");
        p.put("org.quartz.jobStore.tablePrefix", "QRTZ_");
        p.put("org.quartz.jobStore.isClustered", "true");
        p.put("org.quartz.jobStore.clusterCheckinInterval", "20000");
        p.put("org.quartz.dataSource.myDS.driver", "oracle.jdbc.OracleDriver");
        p.put("org.quartz.dataSource.myDS.maxConnections", "5");
        p.put("org.quartz.dataSource.myDS.validationQuery", "select 0 from dual");
        //ENV specific
        p.put("org.quartz.dataSource.myDS.URL", dbUrl);
        p.put("org.quartz.dataSource.myDS.user", dbUser);
        p.put("org.quartz.dataSource.myDS.password", dbPassword);
        qc.setProperties(p);
        return qc;
    }
}
```

# 提示和技巧

*   第一次部署到同一个环境中的两个服务器时，您会看到填充了 qrtz_job_details 和 qrtz_cron_triggers 等表。不管部署两次，每个作业/路线一行。在重新部署**相同版本**时，如果您更改某些内容，该行可能会被替换。请注意，在部署新版本时，将为每个作业/路线添加一个新行。不要担心，quartz 能够跟踪哪一行是**的最新版本**，并且仍然只开始一次工作。
*   首先确保单个本地环境能够工作，并在迁移到集群环境之前填充您的作业存储。
*   直接配置 camel quartz 组件，以确保它通过 quartz 属性文件或 java.util.properties 获得正确的属性。
*   首次部署此配置之前，请在 eap 中清除以下内容。jboss-eap/standalone/tmp、jboss-eap/standalone/data、jboss-eap/standalone/logs，以及 standalone.xml 的部署部分或相应的文件夹和配置文件，用于您正在使用的任何配置。

* * *

**点击这里快速上手 JBoss**[**EAP**](https://developers.redhat.com/products/eap/download/?intcmp=7016000000124dvAAA)**下载。**

*Last updated: August 8, 2017*