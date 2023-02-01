# libssh 中的智能卡支持

> 原文：<https://developers.redhat.com/blog/2020/10/28/smart-cards-support-in-libssh>

在计算机安全中，加密算法的软件实现容易受到*旁路攻击*。这种类型的攻击试图从计算机系统而不是从它正在运行的程序中收集信息。例如，Spectre 和 Meltdown 都是针对现代处理器微体系结构的旁道攻击。微体系结构攻击只是所有旁路攻击的一个子集。还有许多其他人泄露敏感的秘密信息。

能够访问内存中未授权区域的攻击者可以发现私有或敏感信息，包括身份验证机密。随之而来的一个问题是，“我在哪里可以安全地存储我的秘密？”

保护秘密的一种方法是将它们存储在硬件令牌中。硬件令牌在物理上将您的密钥与主机及其运行的应用程序分开。您可以使用存储在智能卡或加密令牌上的密钥对服务器端应用程序进行身份验证。

本文介绍了公钥加密标准#11 ( [PKCS #11](http://docs.oasis-open.org/pkcs11/pkcs11-base/v2.40/os/pkcs11-base-v2.40-os.html) ，您可以使用它来惟一地标识存储在令牌中的对象。我将向您展示如何构建和使用支持 PKCS #11 的 libssh，以及如何使用`curl`通过安全外壳(ssh)协议存储和检索令牌。

## PKCS #11 接口

PKCS #11 提供了一个应用程序编程接口(API ),用于与智能卡之类的设备进行交互，这些设备存储私有加密信息。这种加密设备被称为*令牌*。PKCS #11 使用统一资源标识符(URI)来唯一地标识存储在令牌中的对象。PKCS #11 URI 由[RFC 7512；PKCS # 11 URI 方案](https://tools.ietf.org/html/rfc7512)定义；

```
pkcs11:token=my-token;object=my-object;type=private?pin-value=1234

```

URI 格式定义如下:

*   每个 PKCS #11 URI 都以方案名“*pkcs11:“*开头。
*   方案名称后面是属性-值对，用分号分隔。
*   参数 *token* 代表设备或智能卡的名称。
*   令牌中的存储对象名称由参数 *object* 标识。
*   对象本身可以是私有、公共、证书、数据或密钥类型。
*   参数 *pin-value* 存储访问私钥所需的个人标识号。

**注意**:令牌描述为认证应用提供了存储加密令牌的硬件设备的逻辑视图。

## libssh 中的智能卡支持

[SSH 库，或 libssh](https://api.libssh.org/stable/libssh_tutorial.html) ，是安全外壳(SSH)协议的一个基于库的实现。它支持使用 PKCS #11 URIs 向远程服务器认证用户。目前，PKCS #11 URI 支持仅在 [libssh 主分支](https://gitlab.com/libssh/libssh-mirror)中可用，在 Fedora 中不可用。下一个 libssh 版本(0.9.x)将包括 PKCS #11 URI 支持，这将在 Fedora 中提供。

## 用 PKCS #11 构建和使用 libssh

SSH 库使用 [OpenSSL](https://www.openssl.org) (安全套接字层)作为其加密后端。OpenSSL 定义了一个名为*引擎*的抽象层，它实现了加密原语。它提供了加密功能，称为*密钥加载*，我们用它从智能卡中加载私有和公共密钥。`engine_pkcs11`模块充当 PKCS #11 模块和 OpenSSL 之间的接口。

要构建和使用支持 PKCS #11 的 libssh，请执行以下操作:

1.  启用`cmake`选项:`$ cmake -DWITH_PKCS11_URI=ON`。
2.  用 OpenSSL 构建。
3.  安装并配置 [engine_pkcs11](https://github.com/OpenSC/libp11) 。
4.  插入一个有效的智能卡或者配置一个可以通过 PKCS #11 访问的加密存储库。

libssh 中的遗留函数被扩展为自动检测提供的文件名是文件路径还是 PKCS #11 URI。您可以用 PKCS #11 URIs 替换包含密钥和证书的文件的路径。如果检测到 PKCS #11 URI，则加载并初始化引擎。引擎从 PKCS #11 设备加载对应于 PKCS #11 URI 的私钥或公钥。

**注意**:如果您希望自己使用公钥进行身份验证，请遵循 libssh 文档的“使用公钥进行身份验证”一节中描述的步骤(参见 *[第 2 章:对身份验证的深入了解](https://api.libssh.org/stable/libssh_tutor_authentication.html)* )。

## 使用 PKCS #11 和 libssh 的公钥认证

下面是一个使用 PKCS #11 URIs 的公钥认证的简单示例:

```
int authenticate_pkcs11_URI(ssh_session session)
{
  int rc;
  char priv_uri[1042] = “pkcs11:token=my-token;object=my-object;type=private?pin-value=1234”;

  rc = ssh_options_set(session, SSH_OPTIONS_IDENTITY, priv_uri);
  assert_int_equal(rc, SSH_OK)

  rc = ssh_userauth_publickey_auto(session, NULL, NULL);

  if (rc == SSH_AUTH_ERROR)
  {
    fprintf(stderr, “Authentication with PKCS #11 URIs failed: %s\n”,
      ssh_get_error(session));
    return SSH_AUTH_ERROR;
  }

  return rc;
}

```

您需要做的不是指定存储私钥文件的路径，而是使用`SSH_OPTIONS_IDENTITY`设置 PKCS #11 URI。

## 使用带有 curl 的 PKCS #11 智能卡

像`curl`这样的应用程序使用`libssh`作为底层库，通过 SSH 协议进行通信。在这个例子中，我们使用`curl`连接到一个安全文件传输协议(SFTP)服务器:

```
curl -kvu root: sftp://localhost –key ‘pkcs11:token=my-token;object=my-object;type=private?pin-value=1234’ — testuser

```

我们可以修改上面的命令，使用 PKCS #11 URI 来测试 SSH `testuser`对本地主机的访问。我们将指定相应的 PKCS #11 URI，而不是在`--key`属性中指定私有密钥的路径。

## 结论

本文简要介绍了如何使用 PKCS #11 标准和 libssh 来存储和访问硬件令牌(如智能卡)中的加密私有信息。我将留给你们两个额外的建议。

首先，提供一个特定的 PKCS #11 URI，它只与引擎中的一个插槽相匹配。如果引擎发现多个插槽可能包含所提供的 PKCS #11 URI 引用的私钥，则引擎不会尝试进行身份验证。其次，如果您对 PKCS #11 URIs 使用椭圆曲线数字签名算法(ECDSA ),请确保将公钥和私钥一起导入令牌。与更常用的 RSA 算法不同，ECDSA 公钥不能从私钥派生。

*Last updated: November 4, 2020*