# 使用带有 Red Hat 单点登录/密钥锁的公共证书

> 原文：<https://developers.redhat.com/blog/2019/02/06/using-a-public-certificate-with-red-hat-single-sign-on-keycloak>

当部署[红帽单点登录](https://developers.redhat.com/blog/category/sso/)/key floak 进行测试或概念验证时，大多数用户会选择使用自签名证书，如官方文档中的[所述。](https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.2/html-single/red_hat_single_sign-on_for_openshift/index)

安装说明很简单，但这种自签名证书会在您的 web 浏览器中触发证书错误消息，还会阻止某些客户端(如 Postman)正常工作。

这篇文章解释了如何使用来自[的公共证书，让我们用 Red Hat 单点登录来加密](https://letsencrypt.org/)。

## 您使用的是公共证书吗？

一个简单有效的方法是使用`curl`来知道你是否在使用公共证书。

```
$ curl https://sso.example.test/auth/realms/master
curl: (60) SSL certificate problem: self signed certificate in certificate chain
More details here: https://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
HTTPS-proxy has similar options --proxy-cacert and --proxy-insecure.
```

如果您从`curl`得到这个输出，那么您使用的是一个自签名证书，这将会给您带来麻烦。

继续阅读，了解如何解决这个问题！

## 说明

在本文的剩余部分，我们将关注在 [OpenShift](http://openshift.com/) 上的 Red Hat 单点登录 7.2 安装。

我假设已经安装了 Red Hat 单点登录，正如在官方文档中使用“sso72-x509-*”模板之一所解释的那样。

首先，转到安装了 Red Hat Single Sign-On 的项目:

```
$ oc project sso
```

我们将使用 [Let's Encrypt](https://letsencrypt.org/) 作为认证机构来检索公共证书，而 [Lego](https://github.com/xenolf/lego) 是一个可以与 Let's Encrypt 对话的客户端。它有几个优点，如易于使用，它被包装成一个容器图像。

在当前项目中安装乐高:

```
$ oc new-app --name lego xenolf/lego:latest
$ oc patch dc lego --type=json -p '[{"op": "add", "path": "/spec/template/spec/containers/0/command", "value": ["/bin/sh", "-c", "while :; do sleep 1; done" ]}]'
$ oc expose dc/lego --port=8080

```

要为您的 Red Hat 单点登录实例获取公共证书，我们需要找到该实例的主机名。主机名是 OpenShift 路由的一个属性，它是作为 Red Hat 单点登录安装的一部分创建的。

查询您的 Red Hat 单点登录路由的主机名，并保存它以备后用:

```
$ hostname=$(oc get route sso -o jsonpath='{.spec.host}')

```

到您的 Red Hat 单点登录实例的这个路由需要替换为到 Lego 的临时路由，以便 Let's Encrypt 可以执行 HTTP 挑战。最简单的方法是删除现有路由，并创建一个具有相同主机名的新路由:

```
$ oc delete route sso
$ oc expose service lego --hostname="$hostname" --name=sso

```

您现在可以从正在运行的 Lego pod 中触发证书请求:

```
$ pod=$(oc get pods -o name -l app=lego |head -n1)
$ oc rsh $pod lego --path /tmp/.lego --http --http.port :8080 -d "$hostname" -m your@email.address --accept-tos run
```

第一个命令获取运行 Lego 的 pod 的名称，并相应地设置一个 shell 变量。第二个命令从 Lego 容器中运行`lego`命令。该命令有几个开关:

*   `--path /tmp/.lego`会将生成的证书存储在`/tmp`中。
*   `--http.port :8080`要求乐高监听端口 8080，以接收 [ACME HTTP 挑战](https://letsencrypt.org/how-it-works/)。
*   `--http`启用 HTTP 质询。
*   `-d "$hostname"`在证书请求中设置我们的 Red Hat 单点登录实例的主机名。
*   `-m your@email.address`设置接收续订通知的电子邮件地址(不要忘记设置您的电子邮件地址！).
*   `--accept-tos`表示您阅读并接受了[让我们加密服务条款](https://acme-v01.api.letsencrypt.org/terms)。
*   `run`触发证书请求。

如果您做的一切都是正确的，您应该会看到类似这样的内容:

```
2019/01/22 12:03:55 [INFO] [hostname] acme: Obtaining bundled SAN certificate
2019/01/22 12:03:56 [INFO] [hostname] AuthURL: https://acme-v02.api.letsencrypt.org/acme/authz/[redacted]
2019/01/22 12:03:56 [INFO] [hostname] acme: Could not find solver for: tls-alpn-01
2019/01/22 12:03:56 [INFO] [hostname] acme: Trying to solve HTTP-01
2019/01/22 12:03:56 [INFO] [hostname] Served key authentication
2019/01/22 12:04:01 [INFO] [hostname] The server validated our request
2019/01/22 12:04:01 accept tcp [::]:8080: use of closed network connection
2019/01/22 12:04:01 [INFO] [hostname] acme: Validations succeeded; requesting certificates
2019/01/22 12:04:03 [INFO] [hostname] Server responded with a certificate.

```

恭喜你；你刚刚颁发了你的第一个公共证书！

从 Lego 容器中取出新颁发的证书，并将其存放在安全的地方:

```
$ oc rsync $pod:/tmp/.lego ~/

```

现在，您应该将证书存储在工作站上的个人文件夹中。

```
$ find ~/.lego/certificates
/Users/redhat/.lego/certificates
/Users/redhat/.lego/certificates/<hostname>.key
/Users/redhat/.lego/certificates/<hostname>.issuer.crt
/Users/redhat/.lego/certificates/<hostname>.json
/Users/redhat/.lego/certificates/<hostname>.crt
```

现在，您可以删除临时路由，并用新的 to 路由替换它，以到达 Red Hat Single Sign-On pod。

```
$ oc delete route sso
$ oc create -f - <<EOF
apiVersion: v1
kind: Route
metadata:
  name: sso
spec:
  host: $hostname
  to:
    kind: Service
    name: sso
  tls:
    termination: edge
    key: |-
$(sed 's/^/      /' ~/.lego/certificates/$hostname.key)
    certificate: |-
$(sed 's/^/      /' ~/.lego/certificates/$hostname.crt)
    caCertificate: |-
$(sed 's/^/      /' ~/.lego/certificates/$hostname.issuer.crt)
EOF
```

通过再次运行`curl`命令，确认您的 Red Hat 单点登录实例现在正在使用公共证书:

```
$ curl https://$hostname/auth/realms/master

```

Lego 不需要连续运行，因此在证书更新之间(每 90 天)，您可以缩小其规模:

```
$ oc scale dc/lego --replicas=0

```

## 结论

在交付概念证明或研讨会时，自签名证书总是令人头疼的事情。本文介绍了一种非常实用的方法来为您的 Red Hat 单点登录实例获取有效的公共证书。

有些方面需要改进，以便在更长的时间内或在生产环境中使用。例如，我们需要使用适当的 [Kubernetes](https://developers.redhat.com/blog/category/kubernetes/) 概念、 [Job](https://docs.openshift.com/container-platform/latest/dev_guide/jobs.html) 和 [Cron](https://docs.openshift.com/container-platform/latest/dev_guide/cron_jobs.html) 来运行 Lego，处理证书更新，并使用 DNS 验证挑战(这需要更复杂的设置，但不涉及删除`sso`路由)。

此外，本文还将介绍如何为您的 Red Hat 单点登录实例获取公共证书，该实例已在互联网上公开部署。如果您的实例出于开发目的部署在您的笔记本电脑上，[`mkcert`项目](https://blog.filippo.io/mkcert-valid-https-certificates-for-localhost/)可以帮助您生成适当的证书，并使它们在您的 web 浏览器中受到信任。

尽管如此，我希望这篇文章能给你一些启发，并鼓励你在你的 Red Hat 单点登录系统中使用公共证书。

另一篇有趣的文章是“[使用 Keycloak/Red Hat SSO 使单点登录变得容易](https://developers.redhat.com/blog/2018/03/19/sso-made-easy-keycloak-rhsso/)”

*Last updated: January 21, 2022*