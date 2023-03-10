# Jakarta EE:与 JPA 共同开发的多租户系统，第 1 部分

> 原文：<https://developers.redhat.com/blog/2020/06/15/jakarta-ee-multitenancy-with-jpa-on-wildfly-part-1>

在这个由两部分组成的系列中，我演示了两种多租户方法，其中 [Jakarta 持久性 API (JPA)](https://projects.eclipse.org/projects/ee4j.jpa) 运行在 [WildFly](https://wildfly.org) 上。在本系列的前半部分，您将学习如何使用数据库实现多承租。在第二部分，我将向您介绍使用模式的多承租。我的两个例子都是基于 JPA 和 [Hibernate](http://hibernate.org) 。

因为我已经将重点放在了实现示例上，所以我不会深入探讨多承租的细节，尽管我将从一个简要的概述开始。还要注意，我假设您熟悉使用 JPA 和 Hibernate 的 Java 持久性。

## 多租户架构

[多租户](https://docs.jboss.org/hibernate/orm/4.1/devguide/en-US/html/ch16.html)是一种允许单个应用服务于多个租户(也称为客户端)的架构。尽管多租户架构中的租户访问相同的应用程序，但他们彼此之间是安全隔离的。此外，每个租户只能访问自己的资源。多租户是软件即服务(SaaS)和云计算应用程序的常见架构方法。一般来说，访问 SaaS 的客户端(或租户)访问的是同一个应用程序，但是每个客户端(或租户)都是相互独立的，并且拥有自己的资源。

多租户架构必须隔离每个租户可用的数据。如果一个租户的数据集出现问题，不会影响其他租户。在关系数据库中，我们使用数据库或模式来隔离每个租户的数据。分离数据的一种方法是让每个租户访问自己的数据库或模式。另一个选项是为多个租户划分一个数据库，如果您使用 JPA 和 Hibernate 的关系数据库，这个选项是可用的。在本文中，我主要关注独立数据库和模式选项。我不会演示如何设置分区。

在 WildFly 这样基于服务器的应用程序中，多租户不同于传统方法。在这种情况下，服务器应用程序通过启动连接和准备要使用的数据库来直接处理数据源。客户端应用程序不需要花费时间来打开连接，这提高了性能。另一方面，将 Enterprise JavaBean s(EJB)用于容器管理的事务可能会导致问题。例如，基于服务器的应用程序可能会产生一个错误来提交或回滚应用程序。

## 实现代码

两个接口对于在 JPA 和 Hibernate 中实现多承租至关重要:

*   MultiTenantConnectionProvider 负责将租户连接到他们各自的数据库和服务。我们将使用这个接口和一个租户标识符在不同租户的数据库之间切换。
*   **currenttenantidentifieresolver**负责识别租户。我们将使用这个接口来定义什么被认为是租户(稍后将详细介绍)。我们还将使用这个接口向`MultiTenantConnectionProvider`提供正确的租户标识符。

在 JPA 中，我们使用`persistence.xml`文件配置这些接口。在接下来的小节中，我将向您展示如何使用这两个接口来创建我们的多承租架构所需的前三个类:`DatabaseMultiTenantProvider`、`MultiTenantResolver`和`DatabaseTenantResolver`。

### DatabaseMultiTenantProvider

`DatabaseMultiTenantProvider`是`MultiTenantConnectionProvider`接口的一个实现。该类包含切换到与给定租户标识符匹配的数据库的逻辑。在 WildFly 中，这意味着切换到不同的*数据源*。`DatabaseMultiTenantProvider`类还实现了`ServiceRegistryAwareService`，它允许我们在配置阶段注入服务。

下面是`DatabaseMultiTenantProvider`类的代码:

```
public class DatabaseMultiTenantProvider implements MultiTenantConnectionProvider, ServiceRegistryAwareService{
    private static final long serialVersionUID = 1L;
    private static final String TENANT_SUPPORTED = "DATABASE";
    private DataSource dataSource;
    private String typeTenancy ;

    @Override
    public boolean supportsAggressiveRelease() {
        return false;
    }
    @Override
    public void injectServices(ServiceRegistryImplementor serviceRegistry) {

        typeTenancy = (String) ((ConfigurationService)serviceRegistry
                .getService(ConfigurationService.class))
                .getSettings().get("hibernate.multiTenancy");

        dataSource = (DataSource) ((ConfigurationService)serviceRegistry
                .getService(ConfigurationService.class))
                .getSettings().get("hibernate.connection.datasource");

    }
    @SuppressWarnings("rawtypes")
    @Override
    public boolean isUnwrappableAs(Class clazz) {
        return false;
    }
    @Override
    public <T> T unwrap(Class<T> clazz) {
        return null;
    }
    @Override
    public Connection getAnyConnection() throws SQLException {
        final Connection connection = dataSource.getConnection();
        return connection;

    }
    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {

        final Context init;
        //Just use the multi-tenancy if the hibernate.multiTenancy == DATABASE
        if(TENANT_SUPPORTED.equals(typeTenancy)) {
            try {
                init = new InitialContext();
                dataSource = (DataSource) init.lookup("java:/jdbc/" + tenantIdentifier);
            } catch (NamingException e) {
                throw new HibernateException("Error trying to get datasource ['java:/jdbc/" + tenantIdentifier + "']", e);
            }
        }

        return dataSource.getConnection();
    }

    @Override
    public void releaseAnyConnection(Connection connection) throws SQLException {
        connection.close();
    }
    @Override
    public void releaseConnection(String tenantIdentifier, Connection connection) throws SQLException {
        releaseAnyConnection(connection);
    }
}

```

如您所见，我们调用了`injectServices`方法来填充`datasource`和`typeTenancy`属性。我们使用`datasource`属性从数据源获取一个连接，并使用`typeTenancy`属性确定该类是否支持`multiTenancy`类型。我们调用`getConnection`方法来获得数据源连接。这种方法使用租户标识符来定位和切换到正确的数据源。

### 多 TenantResolver

`MultiTenantResolver`是实现`CurrentTenantIdentifierResolver`接口的抽象类。这个类旨在为所有的`CurrentTenantIdentifierResolver`实现提供一个`setTenantIdentifier`方法:

```
public abstract class MultiTenantResolver implements CurrentTenantIdentifierResolver {

    protected String tenantIdentifier;

    public void setTenantIdentifier(String tenantIdentifier) {
        this.tenantIdentifier = tenantIdentifier;
    } }

```

这个抽象类很简单。我们只使用它来提供`setTenantIdentifier`方法。

### 数据库管理员

`DatabaseTenantResolver`也实现了`CurrentTenantIdentifierResolver`接口。这个类是`MultiTenantResolver`的具体类:

```
public class DatabaseTenantResolver extends MuiltiTenantResolver {

    private Map<String, String> regionDatasourceMap;

    public DatabaseTenantResolver(){
        regionDatasourceMap = new HashMap();
 regionDatasourceMap.put("default", "MyDataSource");
 regionDatasourceMap.put("america", "AmericaDB");
 regionDatasourceMap.put("europa", "EuropaDB");
 regionDatasourceMap.put("asia", "AsiaDB");
    }

    @Override
    public String resolveCurrentTenantIdentifier() {

        if(this.tenantIdentifier != null
 && regionDatasourceMap.containsKey(this.tenantIdentifier)){
 return regionDatasourceMap.get(this.tenantIdentifier);
 }

 return regionDatasourceMap.get("default");

    }

    @Override
    public boolean validateExistingCurrentSessions() {
        return false;
    }

}
```

请注意，`DatabaseTenantResolver`使用一个`Map`为给定的租户定义正确的数据源。在这种情况下，租户是一个区域。还要注意，这个例子假设我们在 WildFly 中配置了数据源`java:/jdbc/MyDataSource`、`java:/jdbc/AmericaDB`、`java:/jdbc/EuropaDB`和`java:/jdbc/AsiaDB`。

## 配置和定义租户

现在我们需要使用`persistence.xml`文件来配置租户:

```
<persistence>
    <persistence-unit name="jakartaee8">

        <jta-data-source>jdbc/MyDataSource</jta-data-source>
        <properties>
            <property name="javax.persistence.schema-generation.database.action" value="none" />
            <property name="hibernate.dialect" value="org.hibernate.dialect.PostgresPlusDialect"/>
            <property name="hibernate.multiTenancy" value="DATABASE"/>
            <property name="hibernate.tenant_identifier_resolver" value="net.rhuanrocha.dao.multitenancy.DatabaseTenantResolver"/>
            <property name="hibernate.multi_tenant_connection_provider" value="net.rhuanrocha.dao.multitenancy.DatabaseMultiTenantProvider"/>
        </properties>

    </persistence-unit>
</persistence>

```

接下来，我们在`EntityManagerFactory`中定义租户:

```
@PersistenceUnit
protected EntityManagerFactory emf;

protected EntityManager getEntityManager(String multitenancyIdentifier){

    final MuiltiTenantResolver tenantResolver = (MuiltiTenantResolver) ((SessionFactoryImplementor) emf).getCurrentTenantIdentifierResolver();
    tenantResolver.setTenantIdentifier(multitenancyIdentifier);

    return emf.createEntityManager();
}

```

注意，我们在创建一个新的`EntityManager`实例之前调用了`setTenantIdentifier`。

## 结论

我已经展示了一个简单的数据库多承租的例子，使用 JPA 和 Hibernate 和 WildFly。有许多方法可以将数据库用于多租户。我的主要观点是向您展示如何实现`CurrentTenantIdentifierResolver`和`MultiTenantConnectionProvider`接口。我已经向您展示了如何使用 JPA 的`persistence.xml`文件来基于这些接口配置所需的类。

记住，对于这个例子，我假设 WildFly 管理数据源和连接池，EJB 处理容器管理的事务。在本系列的第二部分，我将提供类似的多承租介绍，但是使用模式而不是数据库。如果你想深入了解这个例子，你可以[在我的 GitHub 库上找到完整的应用代码和进一步的说明](https://github.com/rhuan080/multitenancyJpaJakartaEE)。

*Last updated: November 12, 2020*