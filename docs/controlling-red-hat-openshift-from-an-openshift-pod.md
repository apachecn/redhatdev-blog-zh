# 从 OpenShift pod 控制 Red Hat OpenShift

> 原文：<https://developers.redhat.com/blog/2019/07/25/controlling-red-hat-openshift-from-an-openshift-pod>

本文解释了如何配置在 [OpenShift](http://developers.redhat.com/openshift/) pod 中运行的 Python 应用程序，以通过`openshift-restclient-python`(open shift Python 客户端)与 Red Hat OpenShift 集群通信。

![](img/c8bc6b09d650ccfe271c7b678a86abdd.png)

## TL；博士；医生

下面的代码示例是一个在 pod 中运行的示例应用程序，它连接到一个 OpenShift 集群并打印一个项目列表:

```
import os
import sys
import yaml
from kubernetes import client, config
from openshift.dynamic import DynamicClient

config.load_incluster_config()
k8s_config = client.Configuration()
k8s_client = client.api_client.ApiClient(configuration=k8s_config)
dyn_client = DynamicClient(k8s_client)

v1_projects = dyn_client.resources.get(api_version="project.openshift.io/v1", kind="Project")

print(v1_projects.get())

```

## 本地运行

从笔记本电脑上使用`openshift-restclient-python`相对容易。这个 OpenShift 动态客户端扩展了标准的 [Kubernetes](http://developers.redhat.com/topics/kubernetes/) Python 客户端。第一步是安装`openshift-restclient-python`，这将引入`kubernetes`依赖项:

```
$ pip install openshift

```

接下来，我们加载一个 Kube 配置。Kubernetes 函数`new_client_from_config()`搜索文件`~/.kube/config`。`new_client_from_config()`函数类似于`load_kube_config()`函数，但也返回一个用于任何 API 对象的`ApiClient`。该任务允许呼叫者同时与多个集群通话。

下面的代码示例使用 OpenShift 动态客户端列出用户可以访问的每个项目:

```
#!/usr/bin/env python3

from kubernetes import client, config
from openshift.dynamic import DynamicClient

k8s_client = config.new_client_from_config()
dyn_client = DynamicClient(k8s_client)

v1_projects = dyn_client.resources.get(api_version='project.openshift.io/v1', kind='Project')

project_list = v1_projects.get()

for project in project_list.items:
    print(project.metadata.name)

```

登录 OpenShift 后在本地运行，如预期的那样:

```
oc login -u user https://ocp.lab.example.com

./cmdlineclient.py
ProjectA
ProjectB

```

## 在 Red Hat OpenShift 中运行

但是，用 OpenShift pod 运行相同的代码将导致一个`TypeError`，如下所示:

```
oc rsh api-gateway-dfs3
cd /opt/app-root/src/
./cmdlineclient.py

Traceback (most recent call last):
  File "./cmdlineclient.py", line 6, in <module>
    k8s_client = config.new_client_from_config()
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 667, in new_client_from_config
    persist_config=persist_config)
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 645, in load_kube_config
    persist_config=persist_config)
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 613, in _get_kube_config_loader_for_yaml_file
    **kwargs)
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 153, in __init__
    self.set_active_context(active_context)
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 173, in set_active_context
    context_name = self._config['current-context']
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 495, in __getitem__
    v = self.safe_get(key)
  File "/opt/app-root/lib/python3.6/site-packages/kubernetes/config/kube_config.py", line 491, in safe_get
    key in self.value):
TypeError: argument of type 'NoneType' is not iterable
```

不幸的是，目前缺少由`openshift-restclient-python`提供的文档。它没有解释如何从 pod 内部连接到 OpenShift 或 Kubernetes。

经过大量的搜索，我在 Kubernetes 文档中找到了一个部分，它指出当从一个 pod 访问 Kube API 时，定位和验证 API 服务器有些不同。他们推荐使用一个官方客户端库，我已经这么做了。这些库应该自动发现 API 服务器并进行身份验证。

Kubernetes 配置库也有功能`load_incluster_config()`。该函数使用环境变量和令牌的组合来验证 API 服务器。推荐的方法是将 pod 与服务帐户相关联。当 pod 启动时，在`/var/run/secrets/kubernetes.io/serviceaccount/token`，服务帐户的令牌被放入该 pod 中每个容器的文件系统树中。

这听起来很简单。但是，在更新`cmdlineclient`之前，我们需要创建一个服务帐户，为它分配一个角色，然后将它与一个 pod 相关联(通过一个部署配置)。以下说明概述了如何使用`oc`客户端来实现这一点:

```
oc create serviceaccount robot

oc policy add-role-to-user admin -z robot

oc patch dc/api-gw --patch '{"spec":{"template":{"spec":{"serviceAccountName": "robot"}}}}'

oc rsh api-gw-9-kzrhn
(app-root) sh-4.2$ ls -al /var/run/secrets/kubernetes.io/serviceaccount/token
lrwxrwxrwx. 1 root root 12 Jul 14 06:13 /var/run/secrets/kubernetes.io/serviceaccount/token -> ..data/token
```

既然我们已经确认了在 pod 中注入了令牌，我们需要更新我们的函数来使用`load_incluster_config()`。然而，记住`new_client_from_config()`返回一个`ApiClient`。我们需要确保在将`ApiClient`传递给 OpenShift 动态客户端之前完成这个更新。另一个未记录的步骤与 OpenShift 动态客户端相关，它需要 Kubernetes `ApiClient`对象中的`client.configuration`对象。

最后，我们还应该确保我们的代码能够在 OpenShift 和我们的笔记本电脑上运行。更新后的`cmdlineclientv2.py`(如下)在调用`load_incluster_config()`之前确定客户端是否在 OpenShift 内运行。它也将退回到读取`~/.kube/config`，这使得程序能够在本地运行:

```
#!/usr/bin/env python3

import os
import sys

import yaml
from kubernetes import client, config
from openshift.dynamic import DynamicClient

# Check if code is running in OpenShift
if "OPENSHIFT_BUILD_NAME" in os.environ:
    config.load_incluster_config()
    file_namespace = open(
        "/run/secrets/kubernetes.io/serviceaccount/namespace", "r"
    )
    if file_namespace.mode == "r":
        namespace = file_namespace.read()
        print("namespace: %s\n" %(namespace))
else:
    config.load_kube_config()

# Create a client config
k8s_config = client.Configuration()

k8s_client = client.api_client.ApiClient(configuration=k8s_config)
dyn_client = DynamicClient(k8s_client)

v1_projects = dyn_client.resources.get(api_version="project.openshift.io/v1", kind="Project")

project_list = v1_projects.get()

for project in project_list.items:
    print("Project Name: %s" % (project.metadata.name))

```

运行`cmdlineclientv2`时，请注意，尽管我们已经为服务帐户分配了`admin`角色，但它只是`ProjectA`名称空间中的`admin`:

```
./cmdlineclientv2.py
namespace: ProjectA

Project Name: ProjectA

```

我希望这篇文章对你有所帮助。欢迎评论和提问。

*Last updated: September 3, 2019*