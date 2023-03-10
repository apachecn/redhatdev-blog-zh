# 在 Red Hat Enterprise Linux 7 上使用 Apache httpd 进行加密

> 原文：<https://developers.redhat.com/blog/2019/08/02/using-lets-encrypt-with-apache-httpd-on-red-hat-enterprise-linux-7>

为您的 web 服务器获取 SSL 证书传统上是一件费力的事情。您需要正确地生成一个奇怪的东西，称为证书签名请求(CSR)，将它提交到您选择的证书颁发机构(CA)的网页，等待他们签名并生成证书，确定将证书放在哪里，以便为您的 web 服务器进行配置——确保您还配置了任何所需的*中间 CA 证书*—然后重新启动 web 服务器。如果你都做对了，那么你需要输入一个日历条目，这样你就能记得在一年后再次经历这个过程。甚至[里面的一些大腕也能搞乱这个过程](https://www.pcworld.com/article/2906216/expired-google-certificate-temporarily-disrupts-gmail-service.html)。

有了像[让我们加密](https://letsencrypt.org/)这样的新 ca，以及一些支持软件，围绕 SSL 证书的繁琐程序将成为过去。这场革命背后的技术是自动证书管理环境(ACME)，这是一个[新的 IETF 标准(RFC 8555)](https://tools.ietf.org/html/rfc8555) 客户端/服务器协议，允许自动获取、部署和更新 TLS 证书。在这个协议中，运行在需要 SSL 证书的服务器上的“代理”将通过 HTTP 与 CA 的 ACME 服务器对话。

在 Red Hat Enterprise Linux 7 服务器上使用 ACME 的一个流行方法是 *certbot* 。Certbot 是一个独立的 ACME 代理，它被配置为开箱即用，可以与 Apache httpd、Nginx 和各种其他 web(和非 web！)服务器。certbot 的作者有一个优秀的[指南，描述了如何在 RHEL7](https://certbot.eff.org/lets-encrypt/centosrhel7-apache.html) 上用 httpd 设置 certbot。

在本教程中，我将展示另一种方法——*mod _ MD*模块——这是一个 ACME 代理，作为 Apache httpd 的模块实现，与 mod_ssl 紧密集成，目前在 Red Hat Enterprise Linux 7 中受支持。mod_md 模块是由 [Stefan Eissing](https://twitter.com/icing) 实现的，他是一位多产的开发人员，也为 httpd 添加了 HTTP/2 支持，并为 Apache 软件基金会做出了贡献，成为了 httpd 版本 2.4.30 以来任何新安装的标准部分。

## 装置

我在 Amazon EC2 中使用一个运行 Red Hat Enterprise Linux 7 的虚拟机，为了开始，我将从[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview?extIdCarryOver=true&intcmp=701f20000012i8UAAQ&sc_cid=701f2000000RtqCAAS)存储库安装 Apache httpd:

```
# yum-config-manager --enable rhui-REGION-rhel-server-rhscl > /dev/null
# yum install -y httpd24 httpd24-mod_ssl httpd24-mod_md
... Installed:
  httpd24.x86_64 0:1.1-18.el7 httpd24-mod_md.x86_64 0:2.4.34-7.el7.1 httpd24-mod_ssl.x86_64 1:2.4.34-7.el7.1

Dependency Installed:
  httpd24-httpd.x86_64 0:2.4.34-7.el7.1 httpd24-httpd-tools.x86_64 0:2.4.34-7.el7.1 httpd24-libcurl.x86_64 0:7.61.1-2.el7 
  httpd24-libnghttp2.x86_64 0:1.7.1-7.el7 httpd24-runtime.x86_64 0:1.1-18.el7

Complete!
#
```

默认情况下不启用软件集合存储库，因此第一步是启用它。注意，`mod_md`的安装就像来自`httpd24`集合的任何其他 httpd 模块一样。

## 配置

现在，要配置服务器。SSL 配置`/opt/rh/httpd24/root/etc/httpd/conf.d/ssl.conf`中需要最小的更改，因此启动您的编辑器并调整默认配置如下:

```
MDomain mytestsslserver.site
ServerAdmin jorton@redhat.com 
<VirtualHost _default:443>
# General setup for the virtual host, inherited from global configuration
#DocumentRoot "/var/www/html"
ServerName mytestsslserver.site:443
...
#SSLCertificateFile /etc/pki/tls/certs/localhost.crt
...
#SSLCertificateKeyFile /etc/pki/tls/private/localhost.key
```

由于`mod_md`将管理该虚拟主机的证书，因此`SSLCertificateFile`和`SSLCertificateKeyFile`必须被移除或注释掉。我还添加了`MDomain`来告诉`mod_md`管理域名，域名必须与`VirtualHost`中使用的名称相匹配，并在`ServerAdmin`中添加了我的电子邮件地址。

如果我们现在启动服务器，我们应该会从 mod_md 得到一些错误:

```
# systemctl start httpd24-httpd
# tail -4 /var/log/httpd24/error_log 
[Mon Jul 22 10:05:09.679997 2019] [mpm_prefork:notice] [pid 5395] AH00163: Apache/2.4.34 (Red Hat) OpenSSL/1.0.2k-fips configured -- resuming normal operations
[Mon Jul 22 10:05:09.680018 2019] [core:notice] [pid 5395] AH00094: Command line: '/opt/rh/httpd24/root/usr/sbin/httpd -D FOREGROUND'
[Mon Jul 22 10:05:11.083123 2019] [md:error] [pid 5397] (70008)Partial results are valid but processing is incomplete: mytestsslserver.site: the CA requires you to accept the terms-of-service as specified in <https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf>. Please read the document that you find at that URL and, if you agree to the conditions, configure "MDCertificateAgreement url" with exactly that URL in your Apache. Then (graceful) restart the server to activate.
[Mon Jul 22 10:05:11.083160 2019] [md:error] [pid 5397] (70008)Partial results are valid but processing is incomplete: AH10056: processing mytestsslserver.site
```

`mod_md`默认使用 Let's Encrypt 服务——但是如果您通过 [`MDCertificateAuthority`指令](https://httpd.apache.org/docs/2.4/mod/mod_md.html#mdcertificateauthority)配置不同的 ACME 服务器，您将在这里得到不同的错误消息。阅读您的 CA 的服务条款后，通过`MDCertificateAgreement`指令表示接受——再次编辑`/opt/rh/httpd24/root/etc/httpd/conf.d/ssl.conf`:

```
MDomain mytestsslserver.site
MDCertificateAgreement https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf ServerAdmin jorton@redhat.com

```

如果您已经阅读了服务条款，您可以添加指令开始并跳过重新加载。我现在重新启动服务器两次，允许证书颁发有短暂的延迟:

```
# systemctl reload httpd24-httpd
# sleep 60
# tail -1 /var/log/httpd24/error_log 
[Mon Jul 22 10:08:19.344969 2019] [md:notice] [pid 5480] AH10059: The Managed Domain mytestsslserver.site has been setup and changes will be activated on next (graceful) server restart.
# systemctl reload httpd24-httpd # tail -1 /var/log/httpd24/ssl_error_log 
[Mon Jul 22 10:09:00.341603 2019] [ssl:info] [pid 5395] AH02568: Certificate and private key mytestsslserver.site:443:0 configured from /opt/rh/httpd24/root/etc/httpd/state/md/domains/mytestsslserver.site/pubcert.pem and /opt/rh/httpd24/root/etc/httpd/state/md/domains/mytestsslserver.site/privkey.pem
```

答对了。最后一条消息告诉我们`mod_md`已经为虚拟主机提供了 SSL 证书和私有密钥。我现在可以通过 https://mytestsslserver.site/的[加载我的新 SSL 站点，得到熟悉的 RHEL“欢迎”页面，并开始向`/opt/rh/httpd24/root/var/www/html/`添加实际内容。](https://mytestsslserver.site/)

## 更多信息

在当前一代的 ACME 协议中，Let's Encrypt 服务器将使用对这个 httpd 服务器的 SSL 请求来确认我是正在请求 SSL 证书的域的所有者。我可以在以下日志中看到这一点:

```
# tail /var/log/httpd24/access_log 
66.133.109.36 - - [22/Jul/2019:10:08:17 +0000] "GET /.well-known/acme-challenge/z2xetCt0LMwehxmGUerh9GFdkg9aqlIXAPWRb_8PEJg HTTP/1.1" 200 87 "-" "Mozilla/5.0 (compatible; Let's Encrypt validation server; +https://www.letsencrypt.org)"
```

为了使这种挑战/响应交换能够工作，您的 SSL 服务器必须可以通过公共网络上的端口 443 访问，而不是受防火墙保护。这意味着您不能将此配置用于私有的内部服务器。有一些方法可以解决这个问题，特别是 DNS 挑战验证，这在 certbot 和 mod_md 的未来版本中都有提供，但这些需要更复杂的配置。

关于使用`mod_md`的更多信息，你可以[阅读上游文档](https://httpd.apache.org/docs/2.4/mod/mod_md.html)。关于红帽软件集合入门的更多信息，[参见发行说明](https://access.redhat.com/documentation/en-us/red_hat_software_collections/3/html/3.2_release_notes/chap-installation#sect-Installation-Subscribe)。

*Last updated: July 29, 2019*