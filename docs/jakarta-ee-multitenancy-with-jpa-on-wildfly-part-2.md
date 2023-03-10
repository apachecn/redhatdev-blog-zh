# Jakarta EE:与 JPA 共同开发的多租户系统，第 2 部分

> 原文：<https://developers.redhat.com/blog/2020/11/12/jakarta-ee-multitenancy-with-jpa-on-wildfly-part-2>

这是关于 WildFly 上的 Jakarta 持久性 API (JPA)的多承租的两部分文章的第二部分。在第 1 部分中，我向您展示了如何使用数据库实现多承租。在第 2 部分中，我将向您展示如何在 [WildFly](https://wildfly.org/) 上使用模式和 [Jakarta 持久性 API](https://projects.eclipse.org/projects/ee4j.jpa) (JPA)来实现多承租。您将学习如何实现 JPA 的`CurrentTenantIdentifierResolver`和`MultiTenantConnectionProvider`接口，以及如何使用 JPA 的`persistence.xml`文件来基于这些接口配置所需的类。

## 实现代码

本文的第一部分提供了 WildFly 上使用 JPA 的多承租的概念性概述，以及一个使用数据库的多承租示例。第二部分将重点转移到使用模式和 JPA 的多承租上。在这种情况下，我假设 WildFly 管理数据源和连接池，EJB([Jakarta Enterprise Beans](https://jakarta.ee/specifications/enterprise-beans/))处理容器管理的事务。

### 多租户的两个接口

正如我在上一篇文章中解释的，两个接口对于在 JPA 和 Hibernate 中实现多承租至关重要:

*   MultiTenantConnectionProvider 接口负责将租户连接到他们各自的数据库和服务。我们将使用这个接口和一个租户标识符在不同租户的数据库之间切换。
*   **currenttenantidentifieresolver**负责识别租户。我们将使用这个接口来定义什么被认为是租户(稍后将详细介绍)，并向`MultiTenantConnectionProvider`提供正确的租户标识符。

接下来，我们将看看实现这些接口的三个类。

### SchemaMultiTenantProvider

`SchemaMultiTenantProvider`是`MultiTenantConnectionProvider`接口的一个实现。该类包含切换到与给定租户标识符匹配的模式的逻辑。

`SchemaMultiTenantProvider`类还实现了`ServiceRegistryAwareService`，它允许我们在配置阶段注入服务。下面是`SchemaMultiTenantProvider`类的代码:

```
public class SchemaMultiTenantProvider implements MultiTenantConnectionProvider, ServiceRegistryAwareService {

    private static final long serialVersionUID = 1L;
    private static final String TENANT_SUPPORTED = "SCHEMA";
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
        resetConnection(connection);// To make sure the connection start using schema/database default.
        return connection;
    }

    @Override
    public Connection getConnection(String tenantIdentifier) throws SQLException {
        //Just use the multitenancy if the hibernate.multiTenancy == SCHEMA
        if(TENANT_SUPPORTED.equals(typeTenancy)) {
            try {
                final Connection connection = getAnyConnection();
                connection.createStatement().execute("SET SCHEMA '" + tenantIdentifier + "'");
                return connection;
            } catch (final SQLException e) {
                 throw new HibernateException("Error trying to alter schema [" + tenantIdentifier + "]", e);
            }
        }

        return getAnyConnection();
    }

    @Override
    public void releaseAnyConnection(Connection connection) throws SQLException {
        //As the Wildfly/JBoss has the Container-Managed Container change the SCHEMA in the end can be dangerous (SET SCHEMA 'public').
        //Thus it just closes the connection.
        connection.close();
    }

    private void resetConnection(Connection connection){
        if(TENANT_SUPPORTED.equals(typeTenancy)) {
            try {
                connection.createStatement().execute("SET SCHEMA 'public'");
            } catch (final SQLException e) {
                throw new HibernateException("Error trying to alter schema [public]", e);
            }
        }
    }

    @Override
    public void releaseConnection(String tenantIdentifier, Connection connection) throws SQLException {
        releaseAnyConnection(connection);
    }

}

```

如您所见，我们调用了`injectServices`方法来填充`datasource`和`typeTenancy`属性。我们使用`datasource`属性从数据源获取一个连接，并使用`typeTenancy`属性确定该类是否支持`multiTenancy`类型。我们调用`getConnection`方法来获得数据源连接。该方法使用租户标识符来定位和切换到正确的模式。

### 多 TenantResolver

`MultiTenantResolver`是一个简单的抽象类，它实现了`CurrentTenantIdentifierResolver`接口。这个类旨在为所有的`CurrentTenantIdentifierResolver`实现提供一个`setTenantIdentifier`方法:

```
public abstract class MultiTenantResolver implements CurrentTenantIdentifierResolver {

    protected String tenantIdentifier;

    public void setTenantIdentifier(String tenantIdentifier) {
        this.tenantIdentifier = tenantIdentifier;
    }
}

```

我们只使用这个类来提供`setTenantIdentifier`方法。

### SchemaTenantResolver

`SchemaTenantResolver`也实现了`CurrentTenantIdentifierResolver`接口。这个类是`MultiTenantResolver`的具体类:

```
public class SchemaTenantResolver extends MuiltiTenantResolver {

    private Map<String, String> userDatasourceMap;

    public SchemaTenantResolver(){
        userDatasourceMap = new HashMap();
        userDatasourceMap.put("default", "public");
        userDatasourceMap.put("username1", "usernameone");
        userDatasourceMap.put("username2", "usernametwo");

    }

    @Override
    public String resolveCurrentTenantIdentifier() {
        if(this.tenantIdentifier != null
                && userDatasourceMap.containsKey(this.tenantIdentifier)){
            return userDatasourceMap.get(this.tenantIdentifier);
        }

        return userDatasourceMap.get("default");
    }

    @Override
    public boolean validateExistingCurrentSessions() {
        return false;
    }

}

```

请注意，`SchemaTenantResolver`使用一个`Map`为给定的租户定义正确的模式。在这种情况下，租户由用户映射。

## 配置和定义租户

现在，我们需要使用 JPA 的`persistence.xml`文件来配置租户:

```
<persistence>
    <persistence-unit name="jakartaee8">

        <jta-data-source>jdbc/MyDataSource</jta-data-source>
        <properties>
            <property name="javax.persistence.schema-generation.database.action" value="none" />
            <property name="hibernate.dialect" value="org.hibernate.dialect.PostgresPlusDialect"/>
            <property name="hibernate.multiTenancy" value="SCHEMA"/>
            <property name="hibernate.tenant_identifier_resolver" value="net.rhuanrocha.dao.multitenancy.SchemaTenantResolver"/>
            <property name="hibernate.multi_tenant_connection_provider" value="net.rhuanrocha.dao.multitenancy.SchemaMultiTenantProvider"/>
        </properties>

    </persistence-unit>
</persistence>

```

我们在 JPA 和 Hibernate `EntityManagerFactory`中定义了租户:

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

本文给出了一个在数据库中使用模式的简单多承租示例。有许多方法可以将数据库用于多租户。我的观点是向您展示如何实现`CurrentTenantIdentifierResolver`和`MultiTenantConnectionProvider`接口。我还向您展示了如何使用 JPA 的`persistence.xml`来基于这两个接口配置所需的类。

请记住，对于这个示例，我假设 WildFly 管理数据源和连接池，并且我们使用企业 beans 进行容器管理的事务。如果你想深入了解这个例子，你可以[在我的 GitHub 库上找到完整的应用代码和进一步的说明](https://github.com/rhuan080/multitenancyJpaJakartaEE)。