# 如何在 OpenStack 13 上安装红帽 OpenShift 3.11

> 原文：<https://developers.redhat.com/blog/2019/04/09/how-to-install-red-hat-openshift-3-11-on-openstack-13>

[Red Hat OpenShift 容器平台](https://developers.redhat.com/products/openshift/overview)是一个平台即服务(PaaS)。它通过 [Kubernetes](https://developers.redhat.com/topics/kubernetes/) 编排和管理容器化的应用程序。虽然 OpenShift 容器平台支持云原生应用，但也支持定制应用。OpenShift 容器平台可以运行在混合云配置上，提供扩展和增长的灵活性。

[Red Hat OpenStack 平台](https://access.redhat.com/products/red-hat-openstack-platform)是一个基础设施即服务(IaaS)。这意味着它是一个基于云的平台，提供虚拟服务器和其他资源。用户可以通过基于 web 的仪表板、命令行工具或 RESTful web 服务来管理它。

如果你在考虑 OpenStack 平台上的 Red Hat OpenShift 容器平台，有几个优点，包括容易增加计算节点的数量，使用动态存储。

在本文中，我将概述在 OpenStack 平台上成功安装 Red Hat OpenShift 容器平台所需的要点。因为我对 OpenStack 的了解有限，所以我向我的同事寻求帮助，在这里我将不涉及太多的 OpenStack 技术细节。

## 先决条件

在开始安装之前，您需要一个符合特定要求的 OpenStack 平台环境。这些主要是认证和红帽订阅需求。以下部分将解决这些问题。

### OpenStack 环境

基本上，您需要按照以下链接设置一个环境:

```
https://docs.openshift.com/container-platform/3.11/install_config/configuring_openstack.html
```

因此，在继续之前，请确保您具备以下条件:

1.  访问启用了所有必需存储库的部署实例，以及访问节点的正确 ssh 密钥
2.  有效的 keystone 身份验证凭据
3.  足够的计算资源来创建您需要的群集，以及任何潜在的增长需求
4.  自动添加新主机的 DNS 服务(就我个人而言，我在这里遇到了一些挑战)

### OpenStack keystone 认证要求

keystone 身份验证有特定的要求。这是为了让 OpenStack 平台云提供商能够通过 OpenStack 平台进行身份验证(主要针对 Cinder 存储)。

主要要求是你的 OpenStack 平台用户和项目存在于同一个域中。如果它们不在同一个域中，安装将会失败(从 OpenShift 容器平台 3.11.69 开始)。

因此，在尝试安装之前，请确保您可以通过以下命令使用项目 ID 和用户进行身份验证:

```
$ openstack \
--os-identity-api-version "3" \
--os-auth-url "https://openstack-default.mydomain:13000/v3" \
--os-username "myorgusername" \
--os-password "mypassword" \
--os-project-id "myprojectid" \
--os-domain-name "myorg" \
server list
```

如果上述命令失败(即使您没有安装任何服务器)，那么您的 OpenShift 容器平台安装将会失败。这是因为 open shift Container Platform cloud provider 无法向用户验证不同域中的项目。

当您运行此命令时，请确保您没有找到 rc 文件(包含上述所有详细信息)，因为您将得到错误的结果。

### 用您的值填充您的 Red Hat OpenShift 清单

以下是一些需要的库存值:

```
$ cat inventory/group_vars/all.yml |grep rhsub
rhsub_server: 'satellite.mydomain'
rhsub_ak: 'openshift'
rhsub_orgid: 'MyOrg'
rhsub_pool: '123456789012345678901234567890'
```

## 调配您的基础堆栈

如果您有所有需要的 OpenStack 设置，运行下面的剧本将创建所有需要的节点(因为您已经在您的`all.yml`文件中指定了它们的编号):

```
$ source openrc.sh
$ ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/openstack/openshift-cluster/provision.yml
```

这将创建您的 OpenShift 节点(根据您的`all.yml`文件)。在本例中，使用了以下设置:

```
openshift_openstack_num_masters: 3
openshift_openstack_num_infra: 3
openshift_openstack_num_cns: 0
openshift_openstack_num_nodes: 3
openshift_openstack_num_etcd: 0
```

### 检查你的基本筹码

完成之前的行动手册后，检查您的动态库存是否已更新:

```
$ soruce openrc.sh
$ /usr/share/ansible/openshift-ansible/playbooks/openstack/inventory.py –list
```

您的动态库存必须在您的`ansible.cfg`文件中设置。您还应该确保所有节点都是可联系的，并且具有正确的 DNS 设置。

我在安装过程中遇到的大多数问题都是由 DNS 引起的。请确保所有 OpenShift 节点的主机名都是`fqdn`主机名。所有节点都必须能够通过 DNS 进行解析。

### 开始安装

要开始安装，请运行以下命令(我个人增加了我的 Ansible 行动手册的超时时间):

```
$ ansible-playbook --timeout=120 /usr/share/ansible/openshift-ansible/playbooks/openstack/openshift-cluster/prerequisites.yml
$ ansible-playbook --timeout=120 /usr/share/ansible/openshift-ansible/playbooks/openstack/openshift-cluster/install.yml
```

## 纵向扩展计算节点

如果您要在不同的平台上扩展 OpenShift 容器平台计算节点，那么您可以按照标准的推荐过程向您的 OpenShift 容器平台集群添加一个新节点。

但是，当您在 OpenStack 平台上运行时，您需要先将这些新节点放入动态清单，然后才能实际执行节点的加入。在 OpenStack 平台上执行后一项任务并不是一项显而易见的任务。

我唯一能够找到相关文档的地方是下面的文件(在部署实例/跳转主机上):

```
/usr/share/ansible/openshift-ansible/playbooks/openstack/configuration.md
```

在这里，我提供了包含在上述文件中的摘录。该摘录引用了在 OpenStack 平台上执行节点纵向扩展所需的确切说明:

```
Section: ### 2\. Scale the Cluster ==>

```
$ ansible-playbook --user openshift \
-i openshift-ansible/playbooks/openstack/scaleup_inventory.py \
-i inventory \
openshift-ansible/playbooks/openstack/openshift-cluster/node-scaleup.yml
```

This will create the new OpenStack nodes, optionally create the DNS records
and subscribe them to RHN, configure the `new_masters`, `new_nodes` and
`new_etcd` groups and run the OpenShift scaleup tasks.

When the playbook finishes, you should have new nodes up and running.

Run `oc get nodes` to verify.

```

在我的例子中，我对从三个计算节点扩展到五个计算节点感兴趣。为此，必须更新`all.yml`文件中的节点计数。

第 1 步:更新`all.yml`文件(此处显示示例):

```
$ cat inventory/group_vars/all.yml | grep openshift_openstack_num_nodes
openshift_openstack_num_nodes: 3
```

收件人:

```
$ cat inventory/group_vars/all.yml | grep openshift_openstack_num_nodes
openshift_openstack_num_nodes: 5
```

第二步:运行 OSP 特有的`node-scaleup.yml`行动手册:

```
$ ansible-playbook \ -i /home/cloud-user/inventory \ -i /usr/share/ansible/openshift-ansible/playbooks/openstack/scaleup_inventory.py \ /usr/share/ansible/openshift-ansible/playbooks/openstack/openshift-cluster/node-scaleup.yml
```

基础节点纵向扩展需要额外的调整，这些调整在`/usr/share/ansible/openshift-ansible/playbooks/openstack/configuration.md`中提供。

我没有尝试纵向扩展主节点，也没有对此进行测试。

## 动态存储器

OpenStack 平台 Cinder 存储可用于配置为利用 OpenStack 平台功能的 OpenShift 容器平台集群。

但是，您应该明白，Cinder 存储是一种读写一次的存储类型，这意味着多个 pod 不能共享同一个存储。在设计您的 Red Hat OpenShift 容器平台集群以及在其上运行的应用程序时，必须考虑这一点。

## 结论

在 OpenStack 平台上安装 Red Hat OpenShift 容器平台提供了许多功能和好处。一个主要优势是能够相对轻松地纵向扩展节点，另一个优势是能够使用 OpenStack 平台 Cinder storage。

尽管关于如何安装基本集群有很好的文档，但是关于向上扩展的文档却很难找到。我在本文中的目的是强调成功安装和扩展 Red Hat OpenShift 集群所需的所有信息。

我没有在这里讨论 OpenStack 平台的技术细节，主要是因为我自己缺乏专业知识，但我确实发现，一旦您拥有了所有正确的信息并启动和运行了所有基础设施服务(主要是 DNS)，在 OpenStack 平台上设置 Red Hat OpenShift 容器平台相对简单。

我感谢我的同事们在完成这一程序中给予的无私帮助。

*Last updated: May 1, 2019*