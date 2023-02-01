# 使用 Ansible 更新容器本机存储的示例

> 原文：<https://developers.redhat.com/blog/2017/11/29/example-using-ansible-update-container-native-storage>

容器原生存储(CNS)在 OpenShift 中作为 pods 实现。这些窗格是根据 OpenShift 内置的模板创建的。在自动安装之后，当使用高级安装程序时，我们希望确保我们有最新的模板和最新的容器。虽然这通常是一个多步骤的手动过程，但 Ansible 脚本使这变得简单多了。

让我们使用一个 Ansible 脚本来完成更新。首先，我们在一个中枢神经系统节点上添加 gluster 通道。接下来，我们将更新 herketi 客户端和 CNS 部署器。在此更新过程中，gluster OpenShift 模板也会更新。我们需要它来更新 OpenShift 中的模板。带有 flat 的 fetch 从 CNS 节点复制它。

在下一步中，我们确保我们在正确的项目中，并删除守护进程集。因为它被设计为在 CNS 上的工作负载之前运行，所以我们删除了 pod 和守护进程集。

以下步骤删除 OpenShift 内部的 Gluster 模板。

在新版本中，节点的标签不同。它使用 local_action 直接在 bastion 主机上运行。您会注意到，该命令将在本地运行多次，节点名称会发生变化。

此时，我们加载新模板并重新创建 CNS 守护进程集。

这方面的原始说明是第 13.3 章中的[这里的](https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.3/html-single/container-native_storage_for_openshift_container_platform/#chap-Documentation-Red_Hat_Gluster_Storage_Container_Native_with_OpenShift_Platform-Upgrade-Gluster_pods)。而这个代码的要点就是[这里的](https://gist.github.com/glennswest/4d59ef7b6529229662bb39e7e4d406ba)。

```
- hosts: cns01
  vars:
    description: "get updated templates for cns"
  tasks:
  - name: add cns channel
    shell: subscription-manager repos --enable=rh-gluster-3-for-rhel-7-server-rpms
  - name: install latest cns-deploy package
    yum: name="cns-deploy" state=latest
  - name: install latest herketi packages
    yum: name="heketi-client" state=latest
  - name: remove cns channel
    shell: subscription-manager repos --disable=rh-gluster-3-for-rhel-7-server-rpms
  - name: Copy templates from cns01 to bastion for /usr/share/heketi/templates
    fetch:
      src:  /usr/share/heketi/templates/glusterfs-template.yaml
      dest: glusterfs-template.yaml
      flat: yes
- hosts: master1
  tasks:
  - name: use gluster project
    shell: oc project glusterfs
  - name: delete daemon set
    command: oc delete ds glusterfs
    ignore_errors: yes
  - name: Delete Old Templates
    shell:  oc delete templates glusterfs
    ignore_errors: yes
- hosts: glusterfs
  tasks:
  - name: relabel nodes
    local_action: command oc label nodes {{ansible_hostname}} storagenode=glusterfs --overwrite
- hosts: localhost
  tasks:
  - name: Use new template
    local_action: command  oc create -f glusterfs-template.yaml
  - name: Wait for distribution of template
    pause:
      seconds: 10
  - name: Deploy the template
    local_action: shell oc process glusterfs | oc create -f -
```

* * *

**利用您的红帽开发者会员资格和** [**免费下载 RHEL**](http://developers.redhat.com/products/rhel/download/) **。**

*Last updated: September 3, 2019*