# 从远程访问 UNIX 套接字。网

> 原文：<https://developers.redhat.com/blog/2019/05/30/accessing-unix-sockets-remotely-from-net>

许多 Linux 服务(如 D-Bus、PostgreSQL、Docker 等。)可以使用 UNIX 套接字进行本地访问。在本文中，我们将展示如何从远程访问这些服务。NET 使用 SSH 端口转发。

## UNIX 套接字

UNIX 域套接字提供了一种在同一主机上运行的进程间交换数据的方式。这种方法也带来了一些安全特性。首先，不可能通过网络访问它们。其次，我们可以识别另一个进程的 userid，并使用它来授权用户。最后，UNIX 域套接字用文件系统中的路径来标识。要访问服务，用户必须拥有该路径的权限。 [SELinux](https://en.wikipedia.org/wiki/Security-Enhanced_Linux) 允许更细粒度的控制。

为了远程访问这些服务，我们可以使用 TCP 套接字而不是 UNIX 套接字来访问它们。然而，这使得服务负责实现认证(识别用户)和加密(确保消息不能被第三方理解)。或者，我们可以使用 SSH 端口转发。

## SSH 端口转发

安全外壳(SSH) 是一种众所周知的在远程机器上运行命令的安全机制。SSH 包括一种针对远程系统进行身份验证的机制，它提供了一个加密的通信通道。

SSH 的一个(可能不太为人所知的)特性是它转发端口的能力。端口转发意味着远程套接字在本地可用。为此，`ssh`客户端程序将打开一个本地套接字，任何到该套接字的连接都将通过安全通道转发，并由 SSH 服务器传递到远程机器上的套接字。

可以通过将`-L`标志传递给`ssh`客户端来设置端口转发:

```
-L [bind_address:]port:host:hostport
-L [bind_address:]port:remote_socket
-L local_socket:host:hostport
-L local_socket:remote_socket

```

如您所见，我们需要指定本地端和远程端。我们可以使用 UNIX 套接字(由文件系统路径标识)或 TCP 套接字(标识为`host:port`)。

例如，要使运行在`mydbserver.org`上的远程 PostgreSQL 服务器在端口`1234`的本地机器上可用，我们可以使用以下命令:

```
ssh -L localhost:1234:/var/run/postgresql/.s.PGSQL.5432 mydbserver.org sleep 10

```

我们的`-L`参数将`localhost:1234`作为本地 TCP 端，将路径`/var/run/postgresql/.s.PGSQL.5432`作为远程 UNIX 套接字端。我们提供了`sleep 10`命令，以便在 10 秒钟内没有 TCP 连接转发的情况下，让`ssh`命令退出。

`ssh`程序不仅在 Linux 上可用，它也是 Windows 10 的一部分。在下一节中，我们将用一个. NET 类来包装它，以提供一种跨平台的方式来设置端口转发。

## 端口转发自。网

[PortForward.cs](https://raw.githubusercontent.com/tmds/dotnet-linux-howto/master/PortForward/PortForward.cs) 提供了一个简单的`PortForward`类来包装`ssh`客户端进行端口转发。

以下示例展示了如何将它与 [Npgsql 包](https://www.nuget.org/packages/Npgsql/)结合使用，以连接到 PostgreSQL 服务器:

```
using (var portForward = await PortForward.ForwardAsync("tmds@192.168.100.169:/var/run/postgresql/.s.PGSQL.5432"))
{
    var connectionString = $"Server={portForward.IPEndPoint.Address};Port={portForward.IPEndPoint.Port};Database=postgres;User ID=tmds";
    using (var connection = new NpgsqlConnection(connectionString))
    {
        connection.Open();
        Console.WriteLine($"PostgreSQL version: {connection.PostgreSqlVersion}");
    }
}

```

在本例中，我们使用用户的预配置私钥。您还可以使用`PortForwardOptions.IdentityFile`显式指定一个密钥文件:

```
var portForward = await PortForward.ForwardAsync(..., o => o.IdentityFile = "mysecretkeyfile");

```

## 结论

在本文中，您了解了 SSH 端口转发如何允许您访问远程 UNIX 套接字。我们已经展示了如何使用`ssh`客户端程序设置端口转发，并在. NET 应用程序中使用它。

*Last updated: May 28, 2019*