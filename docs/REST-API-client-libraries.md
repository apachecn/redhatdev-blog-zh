# 哪里可以找到 OpenShift 的 REST API 客户端库？

> 原文：<https://developers.redhat.com/openshift/REST-API-client-libraries>

作为用户，你通常会通过 web 控制台或`oc`命令行客户端与 OpenShift 交互。当使用这两种方法中的任何一种时，在幕后，他们通过 REST API 端点与 OpenShift 进行对话。

您可以使用 HTTP 客户端(如`curl`)直接访问这个 REST API 端点。

```
#!/bin/sh

SERVER=`oc whoami --show-server`
TOKEN=`oc whoami --show-token`

URL="$SERVER/oapi/v1/users/~"

curl -H "Authorization: Bearer $TOKEN" $URL 
```

除了能够使用 HTTP 客户端(如`curl`)之外，REST API 客户端库也可用于许多不同的编程库。

*   [出发](https://github.com/openshift/client-go)
*   [Java](https://github.com/openshift/openshift-restclient-java)
*   [Python](https://github.com/openshift/openshift-restclient-python)

因为 OpenShift 使用 Kubernetes，所以您也可以使用任何 Kubernetes API 客户端库，但是将被限制为只能使用那些来与 Kubernetes 资源对象和 API 端点进行交互。Kubernetes 客户端不知道 OpenShift 添加的其他资源对象类型和 API 端点。

[在 OpenShift 上开发应用](https://developers.redhat.com/openshift)

**红帽 OpenShift 集装箱平台**

【OpenShift 和 Kubernetes 有什么区别？

[有哪些关于 OpenShift 的书籍？](https://developers.redhat.com/openshift/openshift-books/)

在哪里可以试用 OpenShift，看看它是什么样的？

[如何在自己的电脑上运行 OpenShift 进行开发？](https://developers.redhat.com/openshift/local-openshift/)

[有哪些使用 OpenShift 的托管服务？](https://developers.redhat.com/openshift/hosting-openshift/)

*Last updated: November 19, 2020*