# 使用 Ansible 行动手册和 CodeReady 工作区自动设置研讨会

> 原文：<https://developers.redhat.com/blog/2020/07/03/automate-workshop-setup-with-ansible-playbooks-and-codeready-workspaces>

在 Red Hat，我们为客户、合作伙伴和其他开源开发者举办了许多面对面和虚拟的研讨会。在大多数情况下，研讨会属于“自带设备”类型，因此我们面临一系列硬件和软件设置和企业终端保护方案，以及不同级别的系统知识。

在过去的几年里，我们大量使用了[Red Hat code ready work spaces](https://developers.redhat.com/products/codeready-workspaces/overview)(CRW)。基于 [Eclipse Che](https://www.eclipse.org/che/) ，CodeReady Workspaces 是一个浏览器内 IDE，为大多数开发人员所熟悉，不需要预安装或系统内部知识。你只需要一个浏览器和你的大脑就能接触到这项技术。

我们还为[红帽 Ansible](https://developers.redhat.com/courses/ansible/) 制作了[一套剧本](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)来自动化我们的[夸尔库斯工作室](https://github.com/RedHat-Middleware-Workshops/quarkus-workshop)。虽然行动手册很有用，但对于在 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 上为 [Quarkus](https://quarkus.io) 开发大规模自动化部署 CodeReady 工作区尤其有用。在本文中，我将介绍我们的剧本，并向您展示如何在您自己的自动化工作中使用它们。

## 大规模自动化:概述

虽然我们使用 CodeReady 工作区和本文中介绍的 Ansible 行动手册来自动化我们的 Quarkus 研讨会，但许多公司使用 CodeReady 工作区来大规模自动化新开发人员的入职。在这种情况下，使用 CodeReady 工作区也有助于保护公司的知识产权(即源代码)，并最大限度地减少“在我的机器上工作”的错误借口。

不管你是在举办研讨会还是在进行入职培训，要想让体验尽可能的顺利，需要做大量的准备工作。对于一个研讨会，您需要部署 [Red Hat OpenShift](https://developers.redhat.com/openshift) ，扩展它以满足预期用户数量的需求，并为每个用户安装和配置 CodeReady 工作区。为了获得最佳体验，您还应该“预热”每个工作区，以便在您完成介绍幻灯片时它已经在运行。你还需要安装任何你将作为车间一部分使用的[操作器](https://developers.redhat.com/topics/kubernetes/operators/)。

在接下来的几节中，我将详细介绍这些步骤，以及我们为自动化这些步骤而构建的 Ansible 行动手册。大多数设置都可以应用于创建遵循公司政策和 IT 政策的定制堆栈，同时满足开发人员的需求。

## 安装 OpenShift

OpenShift 用于混合*云*基础设施，因此对于大规模部署，您不必像传统意义上下载 zip 文件、解压缩并在桌面上运行那样“安装”它。你可以用 [CodeReady Containers](https://developers.redhat.com/products/codeready-containers/overview) 进行这种类型的安装，但是当你在一个工作室中支持数十或数百名开发人员时，在本地运行就不是一个选择了。

您可以[在几个不同的公共云和私有云上轻松配置](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.4/html/installing/index) OpenShift。虽然[部署 OpenShift](https://developers.redhat.com/courses/openshift/) 超出了本文的范围，但是我们发现为每五个学生部署一个额外的 OpenShift worker 节点是有用的，其中每个节点有 64 GiB 内存。该设置为每个学生提供了积极的研讨会体验。

对于 Quarkus 研讨会，我们让学生进行原生 Quarkus 构建，这需要额外的内存。每个学生还部署他们自己的 Kafka 集群和一些其他项目。因此，我们只需运行一次研讨会，让所有内容都运行，然后将所有内容相加，以确定每个用户所需的内存量。请记住，对于 CodeReady 工作区，我们使用“每个工作区”的持久卷声明(PVC)策略，其中每个工作区(因此每个用户)都有自己的存储。如果您选择遵循该策略，您将需要确保您有足够的存储空间。你能负担的 CPU 越多越好。

一旦安装了 OpenShift，您将需要创建用户。您可以使用基本的 Linux shell 脚本和`oc` CLI 来覆盖默认的 OpenShift 认证机制，并提供一个包含您的用户(包括管理员用户)的`htpasswd`文件。对于这个 Bash 脚本，您还需要 [`htpasswd`实用程序](https://serverfault.com/questions/259505/how-can-i-install-the-htpasswd-utility-in-red-hat-scientific-linux)和 [`yq`实用程序](https://github.com/mikefarah/yq)(版本 3 或更高版本):

```
#!/bin/bash
NUMUSERS=20
TMPHTPASS=$(mktemp)
for i in {1..$NUMUSERS} ; do
    htpasswd -b ${TMPHTPASS} "user$i" 'somepassword'
done

htpasswd -b ${TMPHTPASS} admin 'adminpassword'

$ oc -n openshift-config delete secret workshop-user-secret

$ oc -n openshift-config create secret generic workshop-user-secret --from-file=htpasswd=${TMPHTPASS}

$ oc -n openshift-config get oauth cluster -o yaml | \
  yq d - spec.identityProviders | \
  yq w - -s htpass-template.yaml | \
  oc apply -f -

sleep 20 # don't shoot the messenger, Operators are "eventually consistent"

$ oc adm policy add-cluster-role-to-user cluster-admin admin

```

与`yq`(版本 3)一起使用的`htpass-template.yaml`模板如下所示:

```
spec.identityProviders[+]:
  name: htpassidp
  type: HTPasswd
  mappingMethod: claim
  htpasswd:
    fileData:
      name: workshop-user-secret

```

使用此模板运行脚本会将新的身份提供者合并到 OpenShift 身份验证流中，以便用户可以登录。您也可以使用 Ansible 来设置这个授权过程，但是我还没有时间来转换它。

## 部署 CodeReady 工作区

我们使用 [CodeReady Workspaces 操作符](https://github.com/redhat-developer/codeready-workspaces-operator)进行安装。为了自动化安装，我们在 Ansible 剧本中使用了一点 Ansible。如果名称空间还不存在，我们使用 [k8s 模块](https://docs.ansible.com/ansible/latest/modules/k8s_module.html)创建一个，连同 OperatorGroup 和 Subscription ( [幂等](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-idempotency)和 all):

```
# create codeready namespace
- name: create codeready namespace
  k8s:
    state: present
    kind: Project
    api_version: project.openshift.io/v1
    definition:
      metadata:
        name: "codeready"
        annotations:
          openshift.io/description: ""
          openshift.io/display-name: "CodeReady Project"

# deploy codeready operator
- name: Create operator subscription for CodeReady
  k8s:
    state: present
    merge_type:
    - strategic-merge
    - merge
    definition: "{{ lookup('file', item ) | from_yaml }}"
  loop:
  - ./files/codeready_operatorgroup.yaml
  - ./files/codeready_subscription.yaml

# wait for CRD to be a thing
- name: Wait for CodeReady CRD to be ready
  k8s_facts:
    api_version: apiextensions.k8s.io/v1beta1
    kind: CustomResourceDefinition
    name: checlusters.org.eclipse.che
  register: r_codeready_crd
  retries: 200
  delay: 10
  until: r_codeready_crd.resources | list | length == 1

# deploy codeready CR
- name: Create CR for CodeReady
  k8s:
    state: present
    merge_type:
    - strategic-merge
    - merge
    definition: "{{ lookup('file', item ) | from_yaml }}"
  loop:
  - ./files/codeready_cr.yaml

# wait for CodeReady to be up
- name: wait for CRW to be running
  uri:
    url: https://codeready-codeready.{{ route_subdomain }}/dashboard/
    validate_certs: false
  register: result
  until: result.status == 200
  retries: "120"
  delay: "15"

```

等待自定义资源定义(CRD)的代码很重要:如果您试图在系统知道 CRD 之前创建一个基于 CRD 的自定义资源(CR ),将会失败。此外，一旦安装了操作器，就需要非零时间来获得该知识。

最后，我们还使用 [uri 模块](https://docs.ansible.com/ansible/latest/modules/uri_module.html)来等待 CodeReady 工作区本身，因为我们接下来要做一些额外的配置。

### 操作员组

OperatorGroup 在`codeready_operatorgroup.yaml`中定义。这很简单，但是要求操作员能够操作:

```
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  generateName: codeready-
  annotations:
    olm.providedAPIs: CheCluster.v1.org.eclipse.che
  name: codeready-operator-group
  namespace: codeready
spec:
  targetNamespaces:
    - codeready

```

### 签署

`codeready_subscription.yaml`中的订阅也是基本的:

```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: codeready-workspaces
  namespace: codeready
spec:
  channel: latest
  installPlanApproval: Automatic
  name: codeready-workspaces
  source: redhat-operators
  sourceNamespace: openshift-marketplace

```

### CheCluster 对象

最后，一旦操作者在 Kube 中注册了它的 CRDs，我们就可以在`codeready_cr.yaml`中创建`CheCluster`对象。创建`CheCluster`启动安装:

```
apiVersion: org.eclipse.che/v1
kind: CheCluster
metadata:
  name: codeready-workspaces
  namespace: codeready
spec:
  server:
    cheImageTag: ''
    cheFlavor: codeready
    devfileRegistryImage: ''
    pluginRegistryImage: ''
    tlsSupport: true
    selfSignedCert: false
    serverMemoryRequest: '2Gi'
    serverMemoryLimit: '6Gi'
    customCheProperties:
      CHE_LIMITS_WORKSPACE_IDLE_TIMEOUT: "0"
  database:
    externalDb: false
    chePostgresHostName: ''
    chePostgresPort: ''
    chePostgresUser: ''
    chePostgresPassword: ''
    chePostgresDb: ''
  auth:
    openShiftoAuth: false
    identityProviderImage: ''
    externalIdentityProvider: false
    identityProviderURL: ''
    identityProviderRealm: ''
    identityProviderClientId: ''
  storage:
    pvcStrategy: per-workspace
    pvcClaimSize: 1Gi
    preCreateSubPaths: true

```

注意内存限制，这是为我们定制的 CodeReady 工作区堆栈中的[容器](https://developers.redhat.com/topics/containers/)调整的。我们也在这里设置了`CHE_LIMITS_WORKSPACE_IDLE_TIMEOUT`。离开一小段时间，当你回来的时候发现你的实验室已经超时，需要刷新(或者要求你重新登录)，这是相当烦人的。当然，这些设置都不应该在生产中使用。

## 调谐键盘锁

[不可能](https://access.redhat.com/documentation/en-us/red_hat_codeready_workspaces/2.1/html/administration_guide/securing-codeready-workspaces_crw#authenticating-to-the-codeready-workspaces-server_authenticating-users)使用 OpenShift 内置的认证机制来预创建和预启动工作区。这样做需要每个用户登录 OpenShift，并将用户的账户信息链接到[红帽单点登录](https://access.redhat.com/products/red-hat-single-sign-on)。(顺便说一下，这就是为什么您会在 CheCluster 资源中看到`openShiftoAuth: false`。)

这个问题的解决方法是在 CodeReady 工作区中创建相同的一组用户，同样使用 Ansible:

```
- name: create codeready users
  include_tasks: add_che_user.yaml
  vars:
    user: "{{ item }}"
  with_list: "{{ users }}"

```

在这个例子中，`users`只是 Ansible 中的一个用户名数组(例如，`[user1, user2, ...]`)。我们遍历并在`add_che_user.yaml`中添加用户，它使用 CodeReady Workspaces REST API 获取 SSO admin 用户的凭证并创建用户:

```
- name: Get codeready SSO admin token
  uri:
    url: https://keycloak-codeready.{{ route_subdomain }}/auth/realms/master/protocol/openid-connect/token
    validate_certs: false
    method: POST
    body:
      username: "{{ codeready_sso_admin_username }}"
      password: "{{ codeready_sso_admin_password }}"
      grant_type: "password"
      client_id: "admin-cli"
    body_format: form-urlencoded
    status_code: 200,201,204
  register: codeready_sso_admin_token

- name: Add user {{ user }} to Che
  uri:
    url: https://keycloak-codeready.{{ route_subdomain }}/auth/admin/realms/codeready/users
    validate_certs: false
    method: POST
    headers:
      Content-Type: application/json
      Authorization: "Bearer {{ codeready_sso_admin_token.json.access_token }}"
    body:
      username: "{{ user }}"
      enabled: true
      emailVerified: true
      firstName: "{{ user }}"
      lastName: Developer
      email: "{{ user }}@no-reply.com"
      credentials:
        - type: password
          value: "{{ workshop_che_user_password }}"
          temporary: false
    body_format: json
    status_code: 201,409

```

本行动手册有几个变量:

*   `route_subdomain`是集群的默认 OpenShift 子域(使用`oc whoami --show-cluster`发现集群)。
*   `workshop_che_user_password`是您的用户所需的密码。
*   `codeready_sso_admin_username/codeready_sso_admin_password`是 CodeReady 工作区使用的 Keycloak 实例的管理员用户名和密码。

为了以编程方式从已部署的 Keycloak 的环境变量中发现 Keycloak 管理员用户名和密码，您可以使用更简单的代码和 [k8s_facts 模块](https://docs.ansible.com/ansible/latest/modules/k8s_facts_module.html):

```
- name: Get codeready keycloak deployment
  k8s_facts:
    kind: Deployment
    namespace: codeready
    name: keycloak
  register: r_keycloak_deployment

- name: set codeready username fact
  set_fact:
    codeready_sso_admin_username: "{{ r_keycloak_deployment.resources[0].spec.template.spec.containers[0].env | selectattr('name','equalto','SSO_ADMIN_USERNAME') |map (attribute='value') | list | first }}"

- name: set codeready password fact
  set_fact:
    codeready_sso_admin_password: "{{ r_keycloak_deployment.resources[0].spec.template.spec.containers[0].env | selectattr('name','equalto','SSO_ADMIN_PASSWORD') |map (attribute='value') | list | first }}"

```

接下来，我们增加了 [SSO 令牌到期时间和 SSO 会话超时时间](https://www.keycloak.org/docs/latest/server_admin/index.html#_timeouts)(同样，这可以让我们避免在研讨会期间令人恼火的注销):

```
- name: Increase codeready access token lifespans
  uri:
    url: https://keycloak-codeready.{{ route_subdomain }}/auth/admin/realms/codeready
    validate_certs: false
    method: PUT
    headers:
      Content-Type: application/json
      Authorization: "Bearer {{ codeready_sso_admin_token.json.access_token }}"
    body:
      accessTokenLifespan: 28800
      accessTokenLifespanForImplicitFlow: 28800
      actionTokenGeneratedByUserLifespan: 28800
      ssoSessionIdleTimeout: 28800
      ssoSessionMaxLifespan: 28800
    body_format: json
    status_code: 204

```

## 预热用户工作区

最后，我们准备好预创建和预热 CRW 工作区:

```
- name: Pre-create and warm user workspaces
  include_tasks: create_che_workspace.yaml
  vars:
    user: "{{ item }}"
  with_list: "{{ users }}"

```

我们将重复一个类似于创建用户工作区的循环。这一次，我们调用`create_che_workspace.yaml`，它使用 CodeReady Workspaces REST API:

```
- name: "Get Che {{ user }} token"
  uri:
    url: https://keycloak-codeready.{{ route_subdomain }}/auth/realms/codeready/protocol/openid-connect/token
    validate_certs: false
    method: POST
    body:
      username: "{{ user }}"
      password: "{{ workshop_che_user_password }}"
      grant_type: "password"
      client_id: "admin-cli"
    body_format: form-urlencoded
    status_code: 200
  register: user_token

- name: Create workspace for {{ user }} from devfile
  uri:
    url: "https://codeready-codeready.{{ route_subdomain }}/api/workspace/devfile?start-after-create=true&namespace={{ user }}"
    validate_certs: false
    method: POST
    headers:
      Content-Type: application/json
      Authorization: "Bearer {{ user_token.json.access_token }}"
    body: "{{ lookup('template', './templates/devfile.json.j2') }}"
    body_format: json
    status_code: 201,409
  register: workspace_def

```

### 关于设计文件

如果你想知道`devfile.json.j2`，它是 [CodeReady devfile](https://developers.redhat.com/blog/2019/12/09/codeready-workspaces-devfile-demystified/) 的 [Ansible Jinja2 模板](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html)。

你可以在这里找到这个例子的 devfile。有趣的部分是:

```
  "components": [
    {
      "id": "redhat/quarkus-java11/latest",
      "type": "chePlugin"
    },

```

请注意，devfile 包括用于工作区的 [Quarkus 插件](https://marketplace.visualstudio.com/items?itemName=redhat.vscode-quarkus)，它提供了 IDE 特性，如自动完成和其他一些小细节:

```
      "image": "image-registry.openshift-image-registry.svc:5000/openshift/quarkus-stack:2.1",

```

这里，我们引用一个 Che 堆栈，该堆栈已经预先生成并作为 ImageStream 部署到 OpenShift 中，使用:

```
apiVersion: image.openshift.io/v1
kind: ImageStream
metadata:
  name: quarkus-stack
  namespace: openshift
spec:
  tags:
  - annotations:
      description: Quarkus stack for Java and CodeReady Workspaces
      iconClass: icon-java
      supports: java
      tags: builder,java
      version: "2.1"
    from:
      kind: DockerImage
      name: quay.io/openshiftlabs/quarkus-workshop-stack:2.1
    name: "2.1"

```

我们使用 Dockerfile 构建了堆栈[，该文件还包含实用程序(`oc`、`kn`、`tkn`和](https://github.com/redhat-cop/agnosticd/blob/development/ansible/roles/ocp4-workload-quarkus-workshop/files/stack.Dockerfile) [GraalVM](https://developers.redhat.com/search?t=+GraalVM) )。它运行了几个测试构建来预填充映像中的 Maven `.m2`存储库，这样用户就不用在每次启动研讨会时下载互联网。通过将该图像预拉入 OpenShift，我们显著减少了工作空间的启动时间。还有[图像拉器](https://www.eclipse.org/che/docs/che-7/caching-images-for-faster-workspace-start/)，我还没用过。它看起来很有希望消除一些这种逻辑。

## 结论

总之，大规模自动化 CodeReady 工作区部署可以显著改善学生体验您的研讨会的方式。预先做尽可能多的事情可以让学生开始学习，而不需要等待安装、热身等等。

本文介绍了我们创建的一些可翻译的行动手册，这些手册旨在通过我们的研讨会自动化和改善用户体验。其他选项包括:

*   部署其他操作员(Strimzi、Jaeger 等)。
*   为工作室创建自定义的 Keycloak 领域。
*   验证研讨会的其他组件是否已正确部署。

请看一下[将 Quarkus 研讨会部署到 OpenShift 4 集群行动手册](https://github.com/redhat-cop/agnosticd/tree/development/ansible/roles/ocp4-workload-quarkus-workshop)。可能有其他的比特你可以使用！此外，如果您对新开发人员的入职示例感兴趣，请查看文章 [*CodeReady Workspaces 交付 Kubernetes-Native IDE*](https://thenewstack.io/codeready-workspaces-delivers-kubernetes-native-ide/) 。

*Last updated: October 18, 2021*