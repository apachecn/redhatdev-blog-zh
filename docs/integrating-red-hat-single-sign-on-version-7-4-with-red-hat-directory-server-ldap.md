# 将 Red Hat 单一登录版本 7.4 与 Red Hat 目录服务器(LDAP)集成

> 原文：<https://developers.redhat.com/blog/2020/12/29/integrating-red-hat-single-sign-on-version-7-4-with-red-hat-directory-server-ldap>

本文描述了[红帽单点登录(SSO)](https://access.redhat.com/products/red-hat-single-sign-on) 与[红帽目录服务器](https://access.redhat.com/products/red-hat-directory-server) 11 (LDAP)的集成。它还说明了如何在 Red Hat Directory Server 和 Red Hat 的单点登录工具之间执行用户同步和组同步。

## 安装红帽目录服务器 11

要安装 Red Hat Directory Server 11，您首先需要一个有效的 Red Hat Directory Server 订阅(可以使用订阅管理器获得)。一旦你有了这个，做以下事情。

### 安装 Red Hat Directory Server 11 软件包

安装 Red Hat Directory Server 11 软件包，包括:

```
# yum module install redhat-ds:11

```

更多详情，请参见章节: [1.1。安装目录服务器软件包。](https://access.redhat.com/documentation/en-us/red_hat_directory_server/11/html/installation_guide/assembly_installing-the-directory-server-packages_installation-guide#proc_installing-the-directory-server-packages_assembly_installing-the-directory-server-packages)

### 创建一个 Red Hat 目录服务器实例

在端口 2389 上创建一个新的 LDAP 实例，使用数据库后缀`dc=example,dc=com`:

```
dscreate interactive
Install Directory Server (interactive mode)

Enter system's hostname [host1.remote.csb]:

Enter the instance name [jdoe]: ds2389

Enter port number []: 2389

Create self-signed certificate database [yes]:

Enter secure port number []: 2636

Enter Directory Manager DN [cn=Directory Manager]:

Enter the Directory Manager password: 
Confirm the Directory Manager Password:

Enter the database suffix (or enter "none" to skip) [dc=jdoe,dc=remote,dc=csb]: dc=example,dc=com

Create sample entries in the suffix [no]: yes
Do you want to start the instance after the installation? [yes]:

Are you ready to install? [no]: yes
Starting installation...
Completed installation for ds2389

```

### 填充 ds2389 实例

使用命令`ldif2db`用现有示例 Example.ldif 填充新实例 ds2389。为此:

1.  停止 ds2389 实例:

```
# dsctl ds2389 stop
Instance "ds2389" has been stopped
```

2.  导入 ldif 示例:

```
# dsctl ds2389 ldif2db userroot /usr/share/dirsrv/data/Example.ldif
ldif2db successful
```

3.  重新启动实例:

```
# dsctl ds2389 start
Instance "ds2389" has been started
```

4.  将实例设置为在计算机启动时自动启动:

```
# systemctl enable dirsrv@ds2389
```

**注**:更多信息，参见红帽目录服务器文档: [1.5。启动和停止目录服务器实例](https://access.redhat.com/documentation/en-us/red_hat_directory_server/11/html/administration_guide/starting_and_stopping-ds)。

### 连接到 LDAP 实例

用 LDAP 浏览器连接到 LDAP 实例，比如 [JXplorer。](http://jxplorer.org/)

要连接的信息有:

```
Connection URL: ldap://localhost:2389
Username: cn=Directory Manager
password: <ds28389-password>
```

[![JXplorer showing an example LDAP entry.](img/80735e7dd83b62079367d7def8918079.png "RH-DS_2389")](/sites/default/files/blog/2020/10/RH-DS_2389.png)

Figure 1: Explore the example ldif in JXplorer.

## 安装和部署 Red Hat SSO 实例

现在是时候安装和部署您的 Red Hat 单点登录实例了。

### 安装 Red Hat SSO

要安装 Red Hat SSO:

1.  下载`rh-sso` zip 发行版。
2.  用以下工具拉开`rh-sso`:

```
unzip rh-sso-<release-number>.zip
```

有关安装的更多详细信息，请参见[入门指南第 2.1 节。安装服务器。](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/getting_started_guide/install-boot#installing_the_server)

### Red Hat SSO 实例部署

要引导 Red Hat 单点登录服务器，请转到服务器发行版的`bin`目录并运行`standalone`引导脚本:

```
cd bin
./standalone.sh
```

*   在网络浏览器中打开[http://localhost:8080/auth](http://localhost:8080/auth)。欢迎页面将显示服务器正在运行。
*   输入用户名和密码以创建初始管理员用户。
*   创建一个 LDAP 领域，如图 2 所示。

[![RH-SSO admin console with ldap realm](img/51c959be9f7b3f3b230829694496fb46.png "img_5fe3024219153")](/sites/default/files/blog/2020/12/img_5fe3024219153.png)

Figure 2: RH-SSO admin console with ldap realm

## 集成 Red Hat 的单点登录工具和 Red Hat 目录服务器

既然您已经安装了 Red Hat 的单点登录工具，那么您就有了一个用于 LDAP 测试的专用领域。执行以下操作来设置集成。

### 联合 LDAP 用户

为了连接到正在运行的 Red Hat Directory Server ds2389 实例，选择**用户联盟**选项卡，并选择 **ldap 提供者** *。*此外，您必须提供以下信息:

*   **卖主** : `Red Hat Directory Server`
*   **用户名 LDAP 属性** : `uid`
*   **RDN LDAP 属性** : `cn`
*   **UUID LDAP 属性** : `nsuniqueid`
*   **用户对象类** : `InetOrgPerson, organizatioalPerson`
*   **连接网址** : `ldap://localhost:2389`
*   **用户 DN** : `ou=people,dc=example,dc=com`
*   **绑定类型** : `simple`
*   **绑定 DN** : `cn=Directory Manager`
*   **搜索范围** : `Subtree`

对于其他字段，选择默认的目的值，如图 3 所示。

[![Configure your LDAP provider for user federation.](img/8292d691139a2060805d80c1cb40a8ca.png "img_5fe30906cd059")](/sites/default/files/blog/2020/12/img_5fe30906cd059.png)

Figure 3: Configure your LDAP provider for user federation.

### 同步单点登录和 LDAP 用户

一旦输入了所有设置，您就可以使用同步设置手动或自动同步用户，如图 4 所示。

[![Under Sync Settings is Cache Settings, which has buttons. Either Synchronize button lets you sync LDAP &amp; SSO users.](img/9e1d034d990868ebec92b21ab880c766.png "ldap_user_stnchronization")](/sites/default/files/blog/2020/10/ldap_user_stnchronization.png)

Figure 4: Synchronize your SSO and LDAP users.

要同步用户:

*   手动:选择**同步所有用户**或**同步变更用户**。如果有大量 LDAP 用户，命令**同步所有用户**可能会很长。
*   自动:根据差值选择**周期性完全同步**或**周期性变化用户同步**。最高效且耗时较少的选择是使用**周期性改变用户同步** *。*

一旦同步完成，您的 LDAP 用户就会在 SSO 级别可见，如图 5 所示。当同步处于活动状态时，新创建的 LDAP 用户也将在此级别进行同步和创建(在下次同步时)。

[![Ldaptest &gt; Manage &gt; Users with the LDAP users now populated](img/da10e2ca7c0427e31ba41568f5ea181a.png "ldap_users")](/sites/default/files/blog/2020/10/ldap_users.png)

Figure 5: Verify that your LDAP users now show up.

## 通过组同步 Red Hat 单点登录和 Red Hat 目录服务器

也可以在 Red Hat 目录服务器和 Red Hat 的单点登录工具之间执行组同步。为了实现这一点，您必须创建一个 group-ldap-mapper 类型的映射器。

#### 配置组 LDAP 映射器

在组-ldap-mapper 部分，提供以下字段:

*   **映射器类型** : `group-ldap-mapper`
*   **LDAP 组 DN** : `ou=groups,dc=example,dc=com`
*   **组名 LDAP 属性** : `cn`
*   **分组对象类** : `groupOfUniqueNames`
*   **成员身份 LDAP 属性** : `uniqueMember`
*   **会员用户 LDAP 属性** : `uid`
*   **LDAP 属性的成员** : `memberOf`

对于所有其他字段，选择默认值，如图 6 所示。

[![Ldaptest &gt; User Federation &gt; Group-ldap-mapper filled in with the specified values.](img/4f07b15b7680b1f3e6a405e4e0a2be42.png "Group_ldap_mapper")](/sites/default/files/blog/2020/10/Group_ldap_mapper.png)

Figure 6: Synchronize your SSO and LDAP groups.

### 3.2 同步您的群组

只能手动执行组同步。您可以使用 **Sync Ldap Group to Keycloak** 按钮从 Red Hat 的单点登录工具到 Ldap 执行组同步，也可以使用 **Sync Keycloak Groups to LDAP** 按钮*从 LDAP 到 Red Hat 的单点登录工具执行组同步。*

在组同步之后，所有的组及其相应的成员都被导入到 SSO 中，如图 7 所示。

[![Ldaptest &gt; Manage &gt; Groups with the LDAP groups now added and populated](img/057509de62c0452f1a45b322894f8d47.png "Group_rh_sso")](/sites/default/files/blog/2020/10/Group_rh_sso.png)

Figure 7: Verify that your LDAP groups now show up.