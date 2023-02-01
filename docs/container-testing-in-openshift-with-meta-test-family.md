# 用元测试族在 OpenShift 中进行容器测试

> 原文：<https://developers.redhat.com/blog/2018/05/22/container-testing-in-openshift-with-meta-test-family>

没有适当的测试，我们不应该装运任何集装箱。我们应该保证容器中给定的服务正常工作。元测试系列 (MTF)就是为此而设计的。

容器可以作为“独立”容器和“协调”容器进行测试。让我们看看如何在 Red Hat [OpenShift](https://www.openshift.com/) 环境下测试容器。本文描述了如何做到这一点，以及需要采取哪些措施。

MTF 是一个建立在现有的[鳄梨](https://avocado-framework.github.io/)和[行为](https://github.com/behave/behave)测试框架上的**极简库**，帮助开发人员快速实现测试自动化和需求。MTF 增加了对测试各种模块工件类型的基本支持和抽象:基于 RPM 的、Docker 映像等等。关于框架以及如何使用它的详细信息，请查看 [MTF 文档](http://meta-test-family.readthedocs.io/en/latest/)。

# 安装 MTF

在开始测试之前，使用`sudo`从官方的 EPEL 库安装 MTF:

```
sudo yum install -y meta-test-family
```

一个 COPR 存储库包含一个不应该在生产环境中使用的 MTF 开发版本。

但是，您可以使用以下命令安装 MTF:

```
dnf copr enable phracek/meta-test-family
dnf install -y meta-test-family
```

要直接从 GitHub 安装 MTF，请运行以下命令:

```
git clone git@github.com:fedora-modularity/meta-test-family.git
cd meta-test-family
sudo python setup.py install
```

现在，您可以开始在 OpenShift 环境中测试容器了。

# 准备 OpenShift 的测试

在本地运行容器非常简单:只需使用`docker run`命令。但这不是你在生产中运行应用程序的方式——这是 OpenShift 的业务。为了确保您的容器编排良好，您应该在相同的环境中测试它们。

请记住，独立环境和协调环境是不同的。独立容器可以通过一个命令轻松执行。管理这样的容器并不容易:您需要弄清楚持久存储、备份、更新、路由和扩展——所有这些都可以通过 orchestrators 免费获得。

OpenShift 环境有自己的特点:安全限制、持久存储逻辑的差异、对无状态 pods 的期望、对更新的支持、多节点环境、本机源到映像支持等等。在这里部署 orchestrator 并不是一件容易的事情。这就是 MTF 支持 OpenShift 的原因:这样你就可以在一个编排好的环境中轻松测试你的容器化应用程序。

在运行和准备 OpenShift 环境之前，您必须为 YAML 格式的 MTF 创建一个测试和配置文件。这两个文件必须在同一个目录中，测试将从这个目录中执行。

# MTF 测试的结构

创建一个包含以下文件的目录:

*   `config.yaml`:MTF 的配置文件
*   `sanity1.py`:MTF 运行的容器测试

# MTF 的配置文件

配置文件如下所示:

```
document: modularity-testing
version: 1
name: memcached
service:
    port: 11211
module:
    openshift:

           start: VARIABLE_MEMCACHED=60 
        container: docker.io/modularitycontainers/memcached

```

下面是 MTF 的 YAML 配置文件中每个字段的解释:

*   `service.port`:服务可用的端口
*   `module.openshift`:仅与 OpenShift 环境相关的配置部分
*   `module.openshift.start` : 将用于 OpenShift 测试的参数
*   `module.openshift.container`:容器的引用，将用于 OpenShift 中的测试

# 测试 memcached 容器

下面是一个容器的`memcached`测试示例:

```
$ cat memcached_sanity.py
import pexpect
from avocado import main
from avocado.core import exceptions
from moduleframework import module_framework
from moduleframework import common

class MemcachedSanityCheck(module_framework.AvocadoTest):

"""
:avocado: enable
"""
def test_smoke(self):
    self.start()
    session = pexpect.spawn("telnet %s %s " % (self.ip_address, self.getConfig()['service']['port']))
    session.sendline('set Test 0 100 4\r\n\n')
    session.sendline('JournalDev\r\n\n')
    common.print_info("Expecting STORED")
    session.expect('STORED')
    common.print_info("STORED was catched")
    session.close()

if __name__ == '__main__':
    main()

```

该测试通过 telnet 在给定的 IP 地址和端口上连接到`memcached`。该端口在 MTF 配置文件中指定。下面几节将详细介绍 IP 地址。

# 为容器测试准备 OpenShift

MTF 可以用`mtf-env-set`命令在本地系统上安装 OpenShift 环境。

```
$ sudo MODULE=openshift OPENSHIFT_LOCAL=yes mtf-env-set
Setting environment for module: openshift
Preparing environment ...
Loaded config for name: memcached
Starting OpenShift
Starting OpenShift using openshift/origin:v3.6.0 ...
OpenShift server started.

The server is accessible via web console at:
https://127.0.0.1:8443

You are logged in as:
User: developer
Password: <any value>

To login as administrator:
oc login -u system:admin

```

`mtf-env-set`命令检查名为`OPENSHIFT_LOCAL`的 shell 变量。如果指定了，该命令将检查是否安装了`origin`和`origin-clients`包。如果没有，它就会安装它们。

在这种情况下，本地机器执行容器测试。如果在远程 OpenShift 实例上测试容器，可以忽略这一步。如果`OPENSHIFT_LOCAL`变量丢失，测试将在由`OPENSHIFT_IP`参数指定的远程 OpenShift 实例上执行(见下文)。

# 容器测试

现在，您可以使用`mtf`命令在本地或远程 OpenShift 实例上测试您的容器。以下命令与前一个命令的唯一区别是命令参数。

在下面的本地测试案例中，`sanity1.py`使用 127.0.0.1 作为`self.ip_address`的值:

```
$ sudo MODULE=openshift OPENSHIFT_USER=developer OPENSHIFT_PASSWORD=developer mtf memcached_sanity.py
```

在下面的远程测试案例中，`sanity1.py`使用`OPENSHIFT_IP`作为`self.ip_address`的值:

```
$ sudo OPENSHIFT_IP=<ip_address> OPENSHIFT_USER=<username> OPENSHIFT_PASSWD=<passwd> mtf memcached_sanity.py
```

然后，从存储给定 OpenShift 实例的配置文件和测试的环境中执行测试。输出如下所示:

```
JOB ID : c2b0877ca52a14c6c740582c76f60d4f19eb2d4d
JOB LOG : /root/avocado/job-results/job-2017-12-18T12.32-c2b0877/job.log
(1/1) memcached_sanity.py:SanityCheck1.test_smoke: PASS (13.19 s)
RESULTS : PASS 1 | ERROR 0 | FAIL 0 | SKIP 0 | WARN 0 | INTERRUPT 0 | CANCEL 0
JOB TIME : 13.74 s
JOB HTML : /root/avocado/job-results/job-2017-12-18T12.32-c2b0877/results.html
$

```

如果你打开`**/**root/avocado/job-results/job-2017-12-18T12.32-c2b0877/job.log` 文件，你会看到类似下面例子的内容。

```
[...snip...]
['/var/log/messages', '/var/log/syslog', '/var/log/system.log'])
2017-12-18 14:29:36,208 job L0321 INFO | Command line: /bin/avocado run --json /tmp/tmppfZpNe sanity1.py
2017-12-18 14:29:36,208 job L0322 INFO |
2017-12-18 14:29:36,208 job L0326 INFO | Avocado version: 55.0
2017-12-18 14:29:36,208 job L0342 INFO |
2017-12-18 14:29:36,208 job L0346 INFO | Config files read (in order):
2017-12-18 14:29:36,208 job L0348 INFO | /etc/avocado/avocado.conf
2017-12-18 14:29:36,208 job L0348 INFO | /etc/avocado/conf.d/gdb.conf
2017-12-18 14:29:36,208 job L0348 INFO | /root/.config/avocado/avocado.conf
2017-12-18 14:29:36,208 job L0353 INFO |
2017-12-18 14:29:36,208 job L0355 INFO | Avocado config:
2017-12-18 14:29:36,209 job L0364 INFO | Section.Key 
[...snip...]

:::::::::::::::::::::::: SETUP ::::::::::::::::::::::::

2017-12-18 14:29:36,629 avocado_test L0069 DEBUG|

:::::::::::::::::::::::: START MODULE ::::::::::::::::::::::::

```

MTF 验证应用程序是否存在于 OpenShift 环境中:。

```
2017-12-18 14:29:36,629 process L0389 INFO | Running 'oc get dc memcached -o json'
2017-12-18 14:29:36,842 process L0479 DEBUG| [stderr] Error from server (NotFound): deploymentconfigs.apps.openshift.io "memcached" not found
2017-12-18 14:29:36,846 process L0499 INFO | Command 'oc get dc memcached -o json' finished with 1 after 0.213222980499s

```

在下一步中，MTF 验证 pod 是否存在于 OpenShift 中:

```
2017-12-18 14:29:36,847 process L0389 INFO | Running 'oc get pods -o json'
2017-12-18 14:29:37,058 process L0479 DEBUG| [stdout] {
2017-12-18 14:29:37,059 process L0479 DEBUG| [stdout] "apiVersion": "v1",
2017-12-18 14:29:37,059 process L0479 DEBUG| [stdout] "items": [],
2017-12-18 14:29:37,059 process L0479 DEBUG| [stdout] "kind": "List",
2017-12-18 14:29:37,059 process L0479 DEBUG| [stdout] "metadata": {},
2017-12-18 14:29:37,059 process L0479 DEBUG| [stdout] "resourceVersion": "",
2017-12-18 14:29:37,059 process L0479 DEBUG| [stdout] "selfLink": ""
2017-12-18 14:29:37,060 process L0479 DEBUG| [stdout] }
2017-12-18 14:29:37,064 process L0499 INFO | Command 'oc get pods -o json' finished with 0 after 0.211796045303s

```

下一步创建一个带有给定标签`mtf_testing`的应用程序，其名称取自`container`标签中的`config.yaml`文件。

```
2017-12-18 14:29:37,064 process L0389 INFO | Running 'oc new-app -l mtf_testing=true docker.io/modularitycontainers/memcached --name=memcached'
2017-12-18 14:29:39,022 process L0479 DEBUG| [stdout] --> Found Docker image bbc8bba (5 weeks old) from docker.io for "docker.io/modularitycontainers/memcached"
2017-12-18 14:29:39,022 process L0479 DEBUG| [stdout]
2017-12-18 14:29:39,022 process L0479 DEBUG| [stdout] memcached is a high-performance, distributed memory object caching system, generic in nature, but intended for use in speeding up dynamic web applications by alleviating database load.
2017-12-18 14:29:39,022 process L0479 DEBUG| [stdout]
2017-12-18 14:29:39,022 process L0479 DEBUG| [stdout] Tags: memcached
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout]
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout] * An image stream will be created as "memcached:latest" that will track this image
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout] * This image will be deployed in deployment config "memcached"
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout] * Port 11211/tcp will be load balanced by service "memcached"
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout] * Other containers can access this service through the hostname "memcached"
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout]
2017-12-18 14:29:39,023 process L0479 DEBUG| [stdout] --> Creating resources with label mtf_testing=true ...
2017-12-18 14:29:39,032 process L0479 DEBUG| [stdout] imagestream "memcached" created
2017-12-18 14:29:39,043 process L0479 DEBUG| [stdout] deploymentconfig "memcached" created
2017-12-18 14:29:39,063 process L0479 DEBUG| [stdout] service "memcached" created
2017-12-18 14:29:39,064 process L0479 DEBUG| [stdout] --> Success
2017-12-18 14:29:39,064 process L0479 DEBUG| [stdout] Run 'oc status' to view your app.
2017-12-18 14:29:39,069 process L0499 INFO | Command 'oc new-app -l mtf_testing=true docker.io/modularitycontainers/memcached --name=memcached' finished with 0 after 2.00025391579s

```

下一步是验证应用程序是否真的在运行，以及它可以在哪个 IP 地址上访问:

```
2017-12-18 14:29:46,201 process L0389 INFO | Running 'oc get service -o json'
2017-12-18 14:29:46,416 process L0479 DEBUG| [stdout] {
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "apiVersion": "v1",
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "items": [
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] {
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "apiVersion": "v1",
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "kind": "Service",
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "metadata": {
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "annotations": {
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] "openshift.io/generated-by": "OpenShiftNewApp"
2017-12-18 14:29:46,417 process L0479 DEBUG| [stdout] },
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "creationTimestamp": "2017-12-18T13:29:39Z",
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "labels": {
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "app": "memcached",
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "mtf_testing": "true"
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] },
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "name": "memcached",
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "namespace": "myproject",
2017-12-18 14:29:46,418 process L0479 DEBUG| [stdout] "resourceVersion": "2121",
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "selfLink": "/api/v1/namespaces/myproject/services/memcached",
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "uid": "7f50823d-e3f7-11e7-be28-507b9d4150cb"
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] },
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "spec": {
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "clusterIP": "172.30.255.42",
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "ports": [
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] {
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "name": "11211-tcp",
2017-12-18 14:29:46,419 process L0479 DEBUG| [stdout] "port": 11211,
2017-12-18 14:29:46,420 process L0479 DEBUG| [stdout] "protocol": "TCP",
2017-12-18 14:29:46,420 process L0479 DEBUG| [stdout] "targetPort": 11211
2017-12-18 14:29:46,420 process L0499 INFO | Command 'oc get service -o json' finished with 0 after 0.213701963425s
2017-12-18 14:29:46,420 process L0479 DEBUG| [stdout] }
2017-12-18 14:29:46,420 process L0479 DEBUG| [stdout] ],
2017-12-18 14:29:46,420 process L0479 DEBUG| [stdout] "selector": {
2017-12-18 14:29:46,420 process L0479 DEBUG| [stdout] "app": "memcached",
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] "deploymentconfig": "memcached",
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] "mtf_testing": "true"
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] },
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] "sessionAffinity": "None",
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] "type": "ClusterIP"
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] },
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] "status": {
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] "loadBalancer": {}
2017-12-18 14:29:46,421 process L0479 DEBUG| [stdout] }
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] }
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] ],
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] "kind": "List",
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] "metadata": {},
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] "resourceVersion": "",
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] "selfLink": ""
2017-12-18 14:29:46,422 process L0479 DEBUG| [stdout] }

```

在最后一个阶段，执行测试。

```
2017-12-18 14:29:46,530 output L0655 DEBUG| Expecting STORED
2017-12-18 14:29:46,531 output L0655 DEBUG| STORED was catched
2017-12-18 14:29:46,632 avocado_test L0069 DEBUG|

:::::::::::::::::::::::: TEARDOWN ::::::::::::::::::::::::

2017-12-18 14:29:46,632 process L0389 INFO | Running 'oc get dc memcached -o json'
2017-12-18 14:29:46,841 process L0479 DEBUG| [stdout] {
2017-12-18 14:29:46,841 process L0479 DEBUG| [stdout] "apiVersion": "v1",
2017-12-18 14:29:46,841 process L0479 DEBUG| [stdout] "kind": "DeploymentConfig",
2017-12-18 14:29:46,841 process L0479 DEBUG| [stdout] "metadata": {

```

在测试结束时，您可以使用命令`oc status`来验证服务是否在 OpenShift 环境中运行:

```
$ sudo oc status
In project My Project (myproject) on server https://127.0.0.1:8443

You have no services, deployment configs, or build configs.
Run 'oc new-app' to create an application.

```

从这个输出中，您可以看到您可以测试一个任意的容器，然后，OpenShift 环境被清除。

# **总结**

正如您在本文中看到的，为容器编写测试非常容易。测试有助于保证容器像 RPM 包一样正常工作。在不久的将来，有计划用 [S2I](https://github.com/openshift/source-to-image) 测试和用 OpenShift 模板测试容器来扩展 MTF 能力。你可以在 [MTF 文档](http://meta-test-family.readthedocs.io/en/latest/)中了解更多。

*Last updated: May 21, 2018*