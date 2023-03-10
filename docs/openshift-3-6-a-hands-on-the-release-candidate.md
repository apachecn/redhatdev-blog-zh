# OpenShift 3.6 -发布候选人(实际操作)

> 原文：<https://developers.redhat.com/blog/2017/07/28/openshift-3-6-a-hands-on-the-release-candidate>

大家好！

今天，我想向您介绍 OpenShift 3.6 的一些特性，同时让您有机会亲身体验这个候选版本。

首先:

1.  这是一个候选版本，我将向您展示的特性被标记为技术预览版，所以仅用于测试目的！
2.  我们不能因为还没有更新的 Minishift 就使用 Minishift。无论如何，我将展示如何使用它的基本 iso-image。
3.  我不想在虚拟机中使用“oc cluster up ”,因为设置虚拟机并运行它会浪费时间。

## 先决条件

这是我们的购物清单，你会在下面找到完成我们的目标所需的所有软件:测试 OpenShift 3.6 RC。

1.  最新的 oc 二进制文件可从 Github 上的[https://Github . com/open shift/origin/releases/tag/v 3 . 6 . 0-RC . 0](https://github.com/openshift/origin/releases/tag/v3.6.0-rc.0)获得。
2.  Docker-Machine:安装了 Docker 的虚拟机！- *"Docker Machine 是一款工具，让你在虚拟主机上安装 Docker 引擎，用 docker-machine 命令管理主机。你可以使用 Machine 在你的本地 Mac 或 Windows box 上，在你的公司网络上，在你的数据中心，或者在云提供商——[https://docs.docker.com/machine/install-machine/](https://docs.docker.com/machine/install-machine/)上创建 Docker 主机。*
3.  虚拟化软件(VirtualBox/Libvirt/KVM/Xhyve)。
4.  足够运行 4GB(或任何其他类似 Minishift 的)虚拟机的 RAM。

**请注意:**

*   如果您以前没有使用过 OpenShift Clients (oc)二进制文件，这并不困难:只需将其解压缩，放在某个地方并运行它。
*   如果你之前没有安装 docker-machine，只需按照之前链接中提供的操作方法:这将很容易！
*   根据您将使用的虚拟化层，您可能需要配置/安装适当的驱动程序，以让 docker-machine 与它一起工作，以下是一些示例:
    *   https://docs.docker.com/machine/drivers/virtualbox/
    *   利比亚/KVM:[https://github.com/dhiltgen/docker-machine-kvm](https://github.com/dhiltgen/docker-machine-kvm)
    *   xhyve:https://github . com/drawing/dock-machine-driver-xhyve
*   在下面的步骤中，我将为我的 Libvirt/KVM 驱动程序使用命令:对不起 mac/win-users！但是你会很容易地适应你的司机的命令，不要担心！所以，看到“ **-kvm-** ”选项的时候要注意编辑命令！
*   所有的命令都可以作为标准用户运行:我们不需要超能力！

## 让我们开始吧:全新的服务目录

如果你在这里，我想你已经配置了你的 docker-machine，不是吗？看前面一段！

正如我前面提到的，我们不能使用 Minishift 二进制文件来构建我们的 Openshift 虚拟机，无论如何，我们可以使用它的 iso 映像作为创建 docker-machine 的源:

> ```
> $ docker-machine create -d "kvm" --kvm-boot2docker-url https://github.com/minishift/minishift-b2d-iso/releases/download/v1.0.2/minishift-b2d.iso --kvm-cpu-count 4 --kvm-memory 4096 --engine-insecure-registry 172.30.0.0/16 openshift
> ```

在前面的命令中，我们正在创建一个名为“Openshift”的虚拟机，从 Minishift [boot2docker](http://boot2docker.io/) 映像开始，使用一些基础设施配置(CPU/RAM)，最重要的是使用 Openshift 的不安全注册表子网配置(172.30.0.0/16)。

您也可以通过运行以下命令来检查前面命令的结果:

> ```
> $  docker-machine ls
> NAME ACTIVE DRIVER STATE URL SWARM DOCKER ERRORS
> openshift - kvm Running tcp://192.168.42.53:2376 v1.12.3
> ```

现在，我们准备启动非常有用的“oc cluster up”命令，该命令提供了许多选项，可以更好地与我们刚刚准备好的 docker-machine 进行交互:

> ```
> $ oc cluster up --docker-machine=openshift --service-catalog=true --public-hostname="$(docker-machine ip openshift).nip.io" --routing-suffix="apps.$(docker-machine ip openshift).nip.io" --use-existing-config=true --host-data-dir='/var/lib/origin/openshift.local.data'
> ```

在前面的命令中，我们只是为我们的 Openshift 平台设置了公共主机名，为路由应用程序设置了通配符 DNS，设置了一些使其持久化的选项，最后是设置全新的**服务目录**的选项。

一旦 OpenShift 平台启动，我们需要以 *system:admin* 的身份登录，然后授予对模板 service broker API 的未经身份验证的访问权限，以便将其用于服务目录:

> ```
> $ oc login -u system:admin
> $ oc adm policy add-cluster-role-to-group system:openshift:templateservicebroker-client system:unauthenticated system:authenticated
> ```

现在我们可以通过进入 Openshift 主页来测试新的服务目录接口了！(你应该可以在前面的*“oc cluster up”*命令的末尾找到它)。

![](img/e6fcd1f092c53e4f4fd42efa1cd7e930.png)

## 向前一步:Ansible Service Broker

此时，我们已经准备好部署最新的特性之一: **Ansible Service Broker** 。

首先，我们必须克隆它的 Github 库:

> ```
> $ git clone https://github.com/openshift/ansible-service-broker
> ```

然后我们必须创建一个全新的项目来部署 Ansible Service Broker 的模板:

> ```
> $ oc new-project ansible-service-broke
> $ oc process -f ansible-service-broker/templates/deploy-ansible-service-broker.template.yaml -p BROKER_IMAGE=ansibleplaybookbundle/ansible-service-broker:latest | oc create -f -
> ```

然后我们应该会看到一些新的 pods 在我们的项目中运行！

> ```
> $ oc get pods
> NAME READY STATUS RESTARTS AGE
> asb-2357364550-4jmcj 1/1 Running 0 1m
> etcd-2338997634-05nz5 1/1 Running 0 1m
> ```

我们到了:Ansible Service Broker 及其 etcd 数据库正在运行！

我们真的离目标很近了。我们需要创建 ASB (Ansible Service Broker)和 Openshift 之间缺少的连接:

> ```
> $ cat << EOF > broker.yaml
> apiVersion: servicecatalog.k8s.io/v1alpha1
> kind: Broker
> metadata:
>   name: ansible-service-broker
> spec:
>   url: https://asb.ansible-service-broker.svc:1338
> EOF
> $ oc create -f broker.yaml
> ```

如果你现在登录到界面，你应该看到一堆全新的模板可用！

![](img/c28f56f16c43dd5edac50dfc3c779be6.png)

以“(APB)”结尾的是 **Ansible 剧本包**的模板！

**请阅读:** 还需要一个步骤，因为 APB 模板使用的一些容器需要“root”权限，所以我们需要为每个经过身份验证的用户启用 ANYUID 安全上下文(最终您可能会将其限制为用户“developer”):

> ```
> $ oc adm policy add-scc-to-group anyuid system:authenticated
> ```

那都是乡亲们！享受你的 OpenShift 3.6 RC 和 **别忘了只把它用于测试目的！**

再见，

## 关于亚历山德罗

Alessandro Arrichiello 是 Red Hat Inc .的解决方案架构师。他从 14 岁开始就对 GNU/Linux 系统充满热情，这种热情一直持续到今天。他使用自动化企业 IT 的工具:配置管理和通过虚拟平台的持续集成。他现在致力于分布式云环境，涉及 PaaS (OpenShift)、IaaS (OpenStack)和流程管理(CloudForms)、容器构建、实例创建、HA 服务管理、工作流构建。

* * *

**如果你知道 Linux 的基本命令，那么下载** [**高级 Linux 命令备忘单**](https://developers.redhat.com/cheat-sheet/advanced-linux-commands-cheatsheet/) **，这个备忘单可以帮助你把你的技能提升到一个新的水平。**

*Last updated: September 3, 2019*