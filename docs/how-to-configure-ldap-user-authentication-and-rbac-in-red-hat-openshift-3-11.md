# 如何在红帽 OpenShift 3.11 中配置 LDAP 用户认证和 RBAC

> 原文：<https://developers.redhat.com/blog/2019/08/02/how-to-configure-ldap-user-authentication-and-rbac-in-red-hat-openshift-3-11>

在本文中，我演示了一种在 [Red Hat OpenShift](http://developers.redhat.com/openshift/) 中配置 LDAP 用户和组同步的系统方法，以及针对这些 LDAP 用户和组的 OpenShift 基于角色的访问控制(RBAC)。遵循这些步骤会使 OpenShift 中 LDAP 用户和组的管理更加容易。我通过展示以下内容来实现这一目标:

*   如何在安装 OpenShift 之前用`ldaptool`验证您的`ldap`参数？
*   如何在 OpenShift 中为特定的 LDAP 组和组织单位启用 LDAP 认证。
*   允许您将 LDAP 组成员同步到 OpenShift 的脚本和命令，从而允许您对特定用户或组应用自定义 OpenShift RBAC 规则。

## 我的假设

出于本文的目的，我假设如下:

*   您有一个工作的 LDAP 服务。
*   允许对您的 OpenShift 环境进行身份验证的唯一两个 ldap 组是`ocp-cluster-admins`和`ocp-cluster-users`，它们与您的 LDAP 树中的`ou=OPENSHIFT`相关联。
*   您已经将`ocp-cluster-admins`分配给用户`ocpadminuser1`，将`ocp-cluster-users`分配给用户`ocpuser1.`

本文展示了为 OpenShift 配置 LDAP 的许多方法之一。因为`ocp-cluster-users`中的不同用户在您的 Red Hat OpenShift 环境中有不同的角色，所以这个任务可以自动化或集中管理，但是这个过程在这里不详细讨论。

请注意，本文有意在各个部分使用混合大小写(大写和小写)来演示 OpenShift 与 LDAP 之间的大小写敏感性。但是，如果您希望尽量减少 OpenShift 区分大小写的问题，请坚持使用小写字母。

## LDAP 详细信息

在开始安装 OpenShift 之前，确定所有的 LDAP 细节。对于本文，我使用以下细节:

```
ldap hostname: myldap.mydomain
ldap bind dn: CN=OPENSHIFT-BU,ou=users,o=MyOrg
ldap bind password: mypassword
ldap ocp admins group DN: cn=ocp-cluster-admins,ou=OPENSHIFT,o=MyOrg
ldap ocp users group DN: cn=ocp-cluster-users,ou=OPENSHIFT,o=MyOrg
```

理想情况下，你所有的秘密凭证都应该通过 [Red Hat Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) 来管理。

## LDAP 参数和凭证测试

为了让您的测试提供可靠的结果，请确认两组中都有用户。在定义 OpenShift 库存参数和用于同步`ldap`组的脚本之前，务必确保所有`ldap`设置都是正确的，与 OpenShift 无关。

我使用`ldapsearch`测试了我的`ldap`连接，详细信息见上一节，我还想确保我能看到我的组中的用户，因为这是 OpenShift 将看到的:

```
$ ldapsearch -x -LLL -D "CN=OPENSHIFT-BU,ou=users,o=MyOrg" -w mypassword -H ldap://myldap.mydomain:389 -b ou=users,o=MyOrg -s sub "(|(memberof=cn=ocp-cluster-admins,ou=OPENSHIFT,o=MyOrg)(memberof=cn=ocp-cluster-users,ou=OPENSHIFT,o=MyOrg))" "CN"

dn: cn=ocp-cluster-admins,ou=OPENSHIFT,o=MyOrg
CN: adminuser1

dn: cn=ocp-cluster-users,ou=OPENSHIFT,o=MyOrg
CN: ocpuser1
```

之后，当您对您的一个用户(例如`ocpuser1`)执行`ldap`搜索时，确定列出了哪些`ldap`字段是很重要的:

```
$ ldapsearch -x -LLL -h myldap.mydomain -D "CN=OPENSHIFT-BU,ou=users,o=MyOrg" -w mypassword -b "ou=users,o=MyOrg" -s sub "cn=ocpuser1"
dn: CN=ocpuser1,ou=users,o=MyOrg
groupMembership: cn=ocp-cluster-users,ou=OPENSHIFT,o=MyOrg
UID: myuid
DESCRIPTION: OpenShift Cluster User 1
SN: ocpuser1
OBJECTCLASS: top
OBJECTCLASS: person
cn: ocpuser1
```

注意，根据您的 LDAP 模式和设计，这些值可能会有所不同。在这种情况下，感兴趣的属性是`DN`、`UID`和`CN`。

## Red Hat OpenShift 库存参数

参考:

*   [使用 LDAP 配置认证](https://docs.openshift.com/container-platform/3.11/install_config/configuring_authentication.html#LDAPPasswordIdentityProvider)

我个人觉得 OpenShift 清单中的 LDAP 连接信息非常混乱。这里，为了清楚起见，我对其进行了解构(注意下面的“属性”部分):

```
openshift_master_identity_providers=[

{
'name': 'myldap', 
'challenge': 'true', 
'login': 'true', 
'kind': 'LDAPPasswordIdentityProvider', 
'attributes': {'DN' : ['DN'], 'UID': ['UID'], 'CN': ['CN']}, 
'bindDN': 'CN=OPENSHIFT-BU,ou=users,o=MyOrg', 
'bindPassword': 'mypassword', 
'ca': '', 
'insecure': 'true', 
'url': 'ldap://myldap.mydomain:389/ou=users,o=MyOrg?CN??
(|
(memberof=cn=ocp-cluster-admins,ou=OPENSHIFT,o=MyOrg)
(memberof=cn=ocp-cluster-users,ou=OPENSHIFT,o=MyOrg)
)'
}

]

```

然后，一旦我清楚了所有的细节，我就删除所有的新行和不必要的空格。代码变为:

```
openshift_master_identity_providers=[{'name': 'myldap', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider', 'attributes': {'DN' : ['DN'], 'UID': ['UID'], 'CN': ['CN']}, 'bindDN': 'CN=OPENSHIFT-BU,ou=users,o=MyOrg', 'bindPassword': 'mypassword', 'ca': '', 'insecure': 'true', 'url': 'ldap://myldap.mydomain:389/ou=users,o=MyOrg?CN??(|(memberof=cn=ocp-cluster-admins,ou=OPENSHIFT,o=MyOrg)(memberof=cn=ocp-cluster-users,ou=OPENSHIFT,o=MyOrg))'}]
```

如果您希望添加`htpasswd`身份验证以备不时之需(例如，仅拥有一个本地管理员用户)，请将其作为另一个身份提供者。例如:

```
openshift_master_identity_providers=[{htpasswd fileds},{ldap fields}]

```

## Red Hat OpenShift 安装后配置

一旦安装完成，之前确定的组中的`ldap`用户可以认证到主 API(使用`oc login`命令),但是默认情况下没有任何访问权限。如果您希望您的用户拥有适当的基于角色的访问权限，您需要执行两个步骤。第一个是将 LDAP 组同步到 OpenShift 组。这个任务需要定期执行，或者每当一个新用户被添加到`ldap`(通过`crontab`或者 CI/CD 管道)时执行。这样做允许 Red Hat OpenShift 将 LDAP 组和用户视为自己的用户。

第二，需要将 RBAC 规则授予 OpenShift 组或用户，从`ldap`开始同步。我手动演示了这一点，但理想情况下，您应该自动完成这一过程。

### 步骤 1:同步`ldap`组

参考:

*   [用 LDAP 同步群组](https://docs.openshift.com/container-platform/3.11/install_config/syncing_groups_with_ldap.html)

在您的主节点上创建包含以下内容的文件`/root/ldap_group_sync.yml`(注释不是文件的一部分，而是用于阐述):

```
# LDAP is case insensitive, but OpenShift is not, so all LDAP parameters have been converted to lower case as per https://access.redhat.com/solutions/3232051 (under "Case Sensitivity")
kind: LDAPSyncConfig
apiVersion: v1
url: ldap://myldapserver:389
insecure: true
ca: ""
bindDN: "cn=openshift-bu,ou=users,o=MyOrg"
bindPassword: "mypassword"
rfc2307:
    groupsQuery:
        baseDN: "ou=openshift,o=MyOrg"
        scope: sub
        filter: (|(cn=ocp-cluster-admins)(cn=ocp-cluster-users))
        derefAliases: never
        timeout: 0
        pageSize: 0
    groupUIDAttribute: dn
    groupNameAttributes: [ cn ]
    groupMembershipAttributes: [ member ]
    usersQuery:
        basedn: "ou=users,o=MyOrg"
        scope: sub
        derefAliases: never
        pageSize: 0
    userUIDAttribute: dn
    userNameAttributes: [ cn ]
    tolerateMemberNotFoundErrors: true
    tolerateMemberOutOfScopeErrors: true

```

运行测试:

```
[root@master ~]# oc adm groups sync --sync-config=/root/ldap_groups.yml
apiVersion: v1
items:
- apiVersion: user.openshift.io/v1
  kind: Group
  metadata:
    annotations:
      openshift.io/ldap.sync-time: 2019-07-21T07:16:1101000
      openshift.io/ldap.uid: cn=ocp-cluster-admins,OU=OPENSHIFT,o=MyOrg
      openshift.io/ldap.url: myldapserver:389
    creationTimestamp: null
    labels:
      openshift.io/ldap.host: myldapserver
    name: ocp-cluster-admins
  users:
  - ocpadminuser1
- apiVersion: user.openshift.io/v1
  kind: Group
  metadata:
    annotations:
      openshift.io/ldap.sync-time: 2019-07-21T07:16:1101000
      openshift.io/ldap.uid: cn=ocp-cluster-users,OU=OPENSHIFT,o=MyOrg
      openshift.io/ldap.url: myldapserver:389
    creationTimestamp: null
    labels:
      openshift.io/ldap.host: myldapserver
    name: ocp-cluster-users
  users:
  - ocpuser1
kind: List
metadata: {}

```

如果您遇到以下错误，您可以[在这里](https://access.redhat.com/solutions/3232051)找到修复方法:

```
For group ignoring member search for entry with dn would search outside of the base dn specified

```

如果您运行上面的命令，这将只是一次演习。为了确保它实际执行用户和组同步，在末尾添加`--confirm`:

```
[root@master ~]# oc adm groups sync --sync-config=/root/ldap_groups.yml --confirm
group/ocp-cluster-admins
group/ocp-cluster-users
```

现在，查看您的小组:

```
# oc get groups
NAME               USERS
ocp-cluster-admins ocpadminuser1
ocp-cluster-users  ocpuser1

```

### 步骤 2:为同步的 LDAP 用户和组创建定制的 OpenShift RBAC 规则

理想情况下，您应该自动化并集中管理这个过程。为不同用户自动分配角色的方法有很多，其中之一就是通过 [Red Hat Ansible](https://www.redhat.com/en/technologies/management/ansible) 。但是，因为我们只有两个用户，手动这样做并不太痛苦。我希望我的`ocp-cluster-admins`组拥有`cluster-admin`角色，我的`ocp-cluster-users`拥有项目编辑角色:

```
root@master ~]# oc adm policy add-cluster-role-to-group cluster-admin ocp-cluster-admins
role "cluster-admin" added: "ocp-cluster-admins"
[root@master ~]# oc adm policy add-role-to-group edit ocp-cluster-users
role "edit" added: "ocp-cluster-users"
```

## 结论

为了确保您的 LDAP 组和用户能够相对顺利地使用 OpenShift RBAC，请使用`ldaptool`验证您的所有 LDAP 细节。此外，最好对所有参数(除了密码)都采用小写，以避免将来由于系统之间区分大小写而导致的问题(例如，`ldap`与 OpenShift)。最后，不同用户角色的自动化是最好的发展方向。

*Last updated: September 3, 2019*