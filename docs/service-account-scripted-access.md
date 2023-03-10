# 如何为脚本访问创建服务帐户？

> 原文：<https://developers.redhat.com/openshift/service-account-scripted-access>

要使用不会过期的会话令牌创建服务帐户，以用于脚本访问，请使用`oc create sa`命令，并将名称传递给服务帐户。

```
$ oc create sa robot
serviceaccount "robot" created 
```

要查看所创建的服务帐户的详细信息，请在服务帐户资源上运行`oc describe`。

```
$ oc describe sa robot
Name:        robot
Namespace:   cookbook
Labels:      <none>
Annotations: <none>

Image pull secrets: robot-dockercfg-vl9qn

Mountable secrets:  robot-token-mhf9x
                    robot-dockercfg-vl9qn

Tokens:             robot-token-4nkdw
                    robot-token-mhf9x 
```

将创建两个访问令牌的秘密。

一个被安装到任何作为该服务帐户运行的容器中，以允许在容器中运行的应用程序在需要时访问 REST API。

第二个在从内部 docker 注册表中提取图像时使用的 docker 配置的单独秘密中引用。

在这两个令牌中，第一个令牌通常会在使用该服务帐户运行的容器中使用，以访问 REST API，当从集群外部访问 REST API 时也会使用它。

要查看访问令牌，请对密码运行`oc describe`。

```
$ oc describe secret robot-token-mhf9x
Name:        robot-token-mhf9x
Namespace:   cookbook
Labels:      <none>
Annotations: kubernetes.io/service-account.name=robot

Type:        kubernetes.io/service-account-token

Data
====
ca.crt:         1070 bytes
namespace:      8 bytes
service-ca.crt: 2186 bytes
token:          eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9... 
```

令牌不会过期。如果您需要撤销访问令牌，您可以使用`oc delete`删除访问令牌的密码，新的密码将被创建。

```
$ oc delete secret robot-token-mhf9x
secret "robot-token-mhf9x" deleted 
```

可以通过对服务帐户运行`oc delete`来删除服务帐户以及与之相关的任何机密。

```
$ oc delete sa robot
serviceaccount "robot" deleted 
```

请注意，默认情况下，服务帐户无权通过 REST API 在项目中做任何事情。您需要向服务帐户授予适当的角色，以使其能够查看或更改任何资源对象。

有关创建和使用服务帐户的更多信息，请参见:

*   [https://docs . open shift . org/latest/dev _ guide/service _ accounts . html](https://docs.openshift.org/latest/dev_guide/service_accounts.html)

*Last updated: November 19, 2020*