# 你好，AMQ 互联世界

> 原文：<https://developers.redhat.com/products/amq/hello-world-amq-interconnect>

## 安装 AMQ 互连路由器

AMQ 互连 1.10 作为一组 RPM 包分发，可通过您的 Red Hat 订阅获得。要获取这些包:

1.  请确保您的订阅已激活，并且您的系统已注册。有关使用客户门户网站激活您的 Red Hat 订阅并为您的系统注册软件包的更多信息，请参见[使用您的订阅](https://access.redhat.com/documentation/en-us/red_hat_jboss_amq/7.0/html-single/using_amq_interconnect/#using_your_subscription)。

2.  订阅所需的存储库:

    ##### 红帽企业版 Linux 7

    `$ sudo subscription-manager repos --enable=amq-interconnect-1-for-rhel-7-server-rpms --enable=amq-clients-2-for-rhel-7-server-rpms`

    ##### 红帽企业版 Linux 8

    `$ sudo subscription-manager repos --enable=amq-interconnect-1-for-rhel-8-x86_64-rpms --enable=amq-clients-2-for-rhel-8-x86_64-rpms`

3.  使用`yum`或`dnf`命令安装`qpid-proton-c`和`python-qpid-proton`包。

    `$ sudo yum install qpid-proton-c python-qpid-proton python-qpid-proton-docs -y`

    AMQ 互联包依赖于这些 Qpid 质子包。

4.  使用`yum`或`dnf`命令安装`qpid-dispatch-router`和`qpid-dispatch-tools`包。

    `$ sudo yum install qpid-dispatch-router qpid-dispatch-console qpid-dispatch-tools -y`

5.  使用`which`命令验证`qdrouterd`可执行文件是否存在。

    `$ which qdrouterd`

`qdrouterd`可执行文件应该位于`/usr/sbin/qdrouterd`。

## 启动路由器服务

1.  要使用默认配置启动路由器，请执行以下操作之一:

    *   要在 Red Hat Enterprise Linux 中将路由器作为服务运行，请输入命令:

        `$ sudo systemctl start qdrouterd.service`

    *   要将路由器作为守护进程运行，请输入命令:

        `$ qdrouterd -d`

        **注意**:要在前台启动路由器，请不要使用`-d`参数。
    *   查看日志以验证路由器状态。

        `$ qdstat --log`

## 发送和接收消息

在发送和接收消息之前，首先需要启动接收方客户端。

### 启动 Python 接收器客户端

要使用 [Python receiver 客户端启动接收器，](https://developers.redhat.com/products/softwarecollections/hello-world#fndtn-python)导航到 Python examples 目录(例如，`/usr/share/proton/`)并运行`simple_recv.py`示例。

`$ cd <install_dir>/examples/python/
$ python simple_recv.py -a 127.0.0.1:5672/examples -m 5`

该命令启动接收器并监听默认地址(`127.0.0.1:5672/examples`)。接收器也被设置为最多接收五条消息。

### 发送消息

启动接收方客户端后，您可以从发送方发送消息，然后消息通过路由器到达接收方。在新的终端窗口中，导航到 Python 示例目录并运行`simple_send.py`示例:

`$ cd <install_dir>/examples/python/
$ python simple_send.py -a 127.0.0.1:5672/examples -m 5`

该命令向默认地址(`127.0.0.1:5672/examples`)发送五条自动生成的消息，然后确认消息已送达并被接收方确认:

`all messages confirmed`

接收方客户端接收消息并显示其内容:

`{u'sequence': int32(1)}
{u'sequence': int32(2)}
{u'sequence': int32(3)}
{u'sequence': int32(4)}
{u'sequence': int32(5)}`

您只是通过 Red Hat A MQ 互连发送和接收消息。请经常访问，查看更多关于 MQ 的教程和其他主题。

*Last updated: February 25, 2021*