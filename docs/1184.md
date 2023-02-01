# 在 RHEL 上使用 Azure CLI 取消分配 Azure 虚拟机

> 原文：<https://developers.redhat.com/blog/2018/04/13/rhel-vm-deallocate-azure-cli>

如果您在 Microsoft Azure 上运行 Red Hat Enterprise Linux server，您可能希望使用虚拟机内部的命令来关闭和解除分配虚拟机，以实现自动化或方便起见。在 Azure 上，如果你使用`shutdown -h`或其他操作系统命令关闭虚拟机，它会停止但不会释放它。停止的虚拟机仍在使用资源，并将继续产生计算费用。为了避免这种情况，本文展示了 VM 如何使用 Azure CLI 2.0 关闭自己并释放资源。

### 安装 Azure CLI 2.0

由于需要 Azure CLI 2.0，请[安装如下](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-yum?view=azure-cli-latest)。

```
$ sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
$ sudo sh -c 'echo -e "[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" &amp;gt; /etc/yum.repos.d/azure-cli.repo'
$ sudo yum install azure-cli
```

### 启用托管服务身份(可选)

[托管服务身份(MSI)](https://docs.microsoft.com/en-us/azure/active-directory/managed-service-identity/overview) 是可选的，但推荐使用，因为它可以避免将密码存储在虚拟机中。如果不想用 MSI，请看最后一节。

要启用 MSI，您可以使用 [Azure CLI 2.0](https://docs.microsoft.com/en-us/azure/active-directory/managed-service-identity/qs-configure-cli-windows-vm#enable-msi-on-an-existing-azure-vm) 、 [Azure Portal](https://docs.microsoft.com/en-us/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm) 或其他管理工具。

我使用 Azure CLI 2.0 启用了 MSI。

```
$ az vm identity assign -g tatanaka-ocp-filetest -n tatanaka-ocp-filetest
```

启用 MSI 后，您必须为启用 MSI 的虚拟机分配适当的角色。

我为[虚拟机贡献者](https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-built-in-roles#virtual-machine-contributor)角色分配了虚拟机本身的范围，因此虚拟机只拥有管理虚拟机本身的特权。为此，使用下面的第一个命令显示虚拟机的资源 ID。使用下面的第二个命令显示虚拟机的服务主体 ID。然后分别用这些值代替下面第三个命令中的`<vm_resource_id>`和`<sp_id>`。

```
$ az vm show -g  -n  --query id
$ az resource list -n  --query [*].identity.principalId --out tsv
$ az role assignment create --assignee  --role 'Virtual Machine Contributor' --scope
```

### 取消分配脚本

现在，您可以在虚拟机中执行 [`az vm deallocate`](https://docs.microsoft.com/ja-jp/cli/azure/vm?view=azure-cli-latest#az-vm-deallocate) 命令，如下所示。

```
$ az login --msi
$ az vm deallocate -g -n
```

但是，您必须在该命令中指定资源组和名称参数。为了避免这种情况，让我们使用 [Azure 实例元数据服务](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service)。

这是释放虚拟机的完整脚本。

```
#!/bin/bash

response=$(curl -H Metadata:true "http://169.254.169.254/metadata/instance/compute?api-version=2017-12-01" -s)
resource_group=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["resourceGroupName"])')
vm_name=$(echo $response | python -c 'import sys, json; print (json.load(sys.stdin)["name"])')

az login --msi
az vm deallocate -g $resource_group -n $vm_name
```

现在，可以登录虚拟机的用户可以取消分配该虚拟机，即使该用户没有 Azure 访问权限。

### 使用密码验证

如果您不使用 MSI，您必须使用密码登录，这意味着您必须在 VM 中存储一个密码用于非交互式释放。为了避免存储微软账户的密码，我强烈建议创建一个 [Azure 服务主体](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)。

您可以创建一个服务主体并分配一个角色，如下所示。

```
$ az ad sp create-for-rbac --name  --password 
$ az vm show -g  -n  --query id
$ az role assignment create --assignee  --role 'Virtual Machine Contributor' --scope
```

在解除分配脚本中，用以下内容替换`--msi`选项:

```
az login --service-principal -u  -p  --tenant
```

*Last updated: January 18, 2022*