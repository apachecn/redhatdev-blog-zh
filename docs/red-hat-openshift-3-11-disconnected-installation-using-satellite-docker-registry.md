# 使用卫星 Docker 注册表的 Red Hat OpenShift 3.11 断开安装

> 原文：<https://developers.redhat.com/blog/2019/04/08/red-hat-openshift-3-11-disconnected-installation-using-satellite-docker-registry>

在本文中，我将讨论使用 Red Hat Satellite 作为本地 Docker 注册表成功实现 Red Hat OpenShift 3.11 断开安装的先决条件和要求，在同事的支持下，我已经能够做到这一点。我还讨论了安装后可能需要的调整。

这项工作基于以下参考资料:

*   [OpenShift 断开安装文档](https://docs.openshift.com/container-platform/3.11/install/disconnected_install.html)
*   [使用 Satellite 6 服务器进行一次断开 OpenShift 容器平台安装](https://blog.openshift.com/using-satellite-6-server-disconnect-openshift-container-platform-install/)

## 先决条件

### 必需的图像

在开始安装之前，您必须同步所有需要的 Docker 映像，并且可以从所有节点访问。

您不需要同步所有映像(总共 100 多个)，根据您的要求，您将需要:

*   red Hat open shift Container Platform 基础架构组件映像，这是您的基本安装所需的，总共有 55 个映像
*   red Hat open shift Container Platform 组件映像为可选组件，这是日志记录和监控所必需的，共有 26 个映像
*   Red Hat 认证的源到图像(S2I)生成器图像，这是您的 OpenShift 应用程序所需的，总共 29 个图像

此外，如果您确切地知道您的安装需要哪些映像(通过检查您在现有环境中拥有的所有 Docker 映像)，您可以只预同步那些映像。

### 确定红帽 OpenShift 3.11 版本

在此安装时，使用的是版本 3 . 11 . 69；但是，您应该安装可用于错误修复和功能的最新版本。

这应该反映在您的库存中，如下所示:

```
openshift_release: v3.11
openshift_image_tag: v3.11.69
openshift_pkg_version: -3.11.69
```

### 红帽卫星设置

在 Red Hat 卫星服务器上，在组织内创建一个产品，您的 Red Hat OpenShift 节点将注册到该产品

```
hammer product create --name "ocp311" --organization "MyOrg"
```

然后，您需要为每个图像创建一个存储库。这使得 Red Hat Satellite 的行为与其他注册表略有不同，因为 Red Hat Satellite 会将此映像与上游服务器(registry.redhat.io)同步，包括其所有版本。例如，要为 openshift3/ose-node 创建产品，请使用以下命令:

```
hammer repository create --name "openshift3/ose-node" --content-type "docker" --url "http://registry.redhat.io/" --docker-upstream-name "openshift3/ose-node" --product "ocp311" --organization "MyOrg"
```

创建所有存储库后，您可以使用 GUI 或通过命令行进行同步:

```
hammer product synchronize --name "ocp311" --organization "MyOrg"
```

确保将产品添加到 OpenShift 将注册访问的内容视图中。

Red Hat Satellite 服务器实现的 Docker 注册表与传统的 Docker 注册表略有不同。这意味着 Docker 图像的 URL 将从:

```
registry.redhat.io/openshift3/ose-pod:v3.11.69
```

到

```
satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-pod:v3.11.69
```

如果您不告诉 Red Hat OpenShift 安装程序此更改，安装将会失败。我将在下一节提供进一步的说明。

测试图像拉取:

```
docker pull satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-pod:v3.11.69
```

## Red Hat OpenShift 可变库存修改

为了让安装程序能够使用 Red Hat Satellite 上的 Docker 注册表，请按如下方式重构`oreg_url`:

```
oreg_url: satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-${component}:${version}
openshift_disable_check: "docker_image_availability"
openshift_docker_insecure_registries: "satellite.mydomain:5000"
openshift_docker_additional_registries: "satellite.mydomain:5000"
openshift_examples_modify_imagestreams: True
```

将`oreg_test_url`设置为假:

```
oreg_test_login: false
```

确保`oreg_auth`值被注释掉。因为不需要身份验证，所以不将它们注释掉会导致问题:

```
# oreg_auth_user: '' # our registry doesn't require authentication. 
# oreg_auth_password: '' # our registry doesn't require authentication.

```

附加库存变量:

```
openshift_examples_registryurl: "{{ oreg_url | default(openshift_hosted_images_dict[openshift_deployment_type]) }}"
registry_host: "{{ openshift_examples_registryurl.split('/')[0] if '.' in openshift_examples_registryurl.split('/')[0] else '' }}"
```

此外，模式`"MyOrg-ocp311-openshift3_ose-${component}:${version}`对 etcd 无效；因此，您必须指定确切的 etcd docker 映像 URL:

```
osm_etcd_image: satellite.mydomain:5000/MyOrg-ocp311-rhel7_etcd:3.2.22
```

## 安装后配置

根据您希望在 OpenShift 集群中使用的应用程序，您可能需要相应地修改图像流。这里就不深究这些细节了。但是，某些任务可能仍会尝试从在线注册表中提取图像来执行其功能，如诊断。

因此，要运行诊断程序，可以将以下映像重新标记为“欺骗”诊断程序，这样它就不会试图从在线注册表中提取映像:

```
ansible nodes --timeout=120 -m shell -b -a 'docker pull satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-deployer:v3.11.69'
ansible nodes --timeout=120 -m shell -b -a 'docker tag satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-control-plane:v3.11.69 registry.redhat.io/openshift3/ose-control-plane:v3.11.69'
ansible nodes --timeout=120 -m shell -b -a 'docker tag satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-control-plane:v3.11.69 registry.redhat.io/openshift3/ose-control-plane:v3.11'

ansible nodes --timeout=120 -m shell -b -a 'docker pull satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-node:v3.11.69'
ansible nodes --timeout=120 -m shell -b -a 'docker tag satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-node:v3.11.69 registry.redhat.io/openshift3/ose-node:v3.11.69'
ansible nodes --timeout=120 -m shell -b -a 'docker tag satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-node:v3.11.69 registry.redhat.io/openshift3/ose-node:v3.11'

ansible nodes --timeout=120 -m shell -b -a 'docker pull satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-deployer:v3.11.69'
ansible nodes --timeout=120 -m shell -b -a 'docker tag satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-deployer:v3.11.69 registry.redhat.io/openshift3/ose-deployer:v3.11.69'
ansible nodes --timeout=120 -m shell -b -a 'docker tag satellite.mydomain:5000/MyOrg-ocp311-openshift3_ose-deployer:v3.11.69 registry.redhat.io/openshift3/ose-deployer:v3.11'

```

然后，您应该能够运行:

```
oc adm diagnostics
```

## 结论

要在断开连接的模式下安装您的集群，本文中的说明将帮助您入门。但是，根据您可能使用的应用程序，可能会遇到丢失图像或引用在线图像的情况。在这种情况下，请留意这种可能性，并考虑更新应用程序的部署或构建配置中的引用，或者仅作为临时措施重新标记 Docker 映像(只是为了排除映像不可访问的错误)。

*Last updated: April 3, 2019*