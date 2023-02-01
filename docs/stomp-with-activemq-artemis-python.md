# 通过 Apache ActiveMQ Artemis 代理使用 STOMP 协议

> 原文：<https://developers.redhat.com/blog/2018/06/14/stomp-with-activemq-artemis-python>

在本文中，我们将在 [Apache ActiveMQ Artemis](https://activemq.apache.org/artemis/) 代理中使用一个基于 Python 的消息客户端来连接和订阅一个具有[持久](https://developers.redhat.com/blog/2016/08/10/persistence-vs-durability-in-messaging/)订阅的主题。我们将使用基于文本的 [STOMP](https://stomp.github.io/) 协议来连接和订阅代理。STOMP 客户端可以与任何 STOMP 消息代理通信，以提供多种语言、平台和代理之间的消息互操作性。

如果你需要温习消息传递中持久性和持久性之间的差异，可以看看玛丽·科克伦关于 developers.redhat.com/blog.的文章

类似的过程可以用在红帽 AMQ 7 号上。红帽 AMQ 7 中的代理基于 Apache ActiveMQ Artemis 项目。更多信息参见 developers.redhat.com 上的[概述](https://developers.redhat.com/products/amq/overview/)。

## **设置项目**

在下面的例子中，我们使用一个客户机来发布和订阅一个主题。你可以在我的[个人 GitHub 回购](https://github.com/1984shekhar/Artemis_POC/tree/master/python_stomp_example)找到代码。我们有两个 [receiver_queue.py](https://github.com/1984shekhar/Artemis_POC/blob/master/python_stomp_example/receiver_queue.py) 和 [receiver_topic.py](https://github.com/1984shekhar/Artemis_POC/blob/master/python_stomp_example/receiver_topic.py) Python 消息客户端。`receiver_queue.py`是一个基于 STOMP 协议的 Python 客户端，用于到代理的点对点(队列)连接，`receiver_topic.py`是一个基于 STOMP 协议的 Python 客户端，用于对代理的主题进行持久订阅。

代码如下:

```
import time
import sys

import stomp

class MyListener(stomp.ConnectionListener):
 def on_error(self, headers, message):
 print('received an error "%s"' % message)
 def on_message(self, headers, message):
 print('received a message "%s"' % message)
hosts = [('localhost', 61616)]

conn = stomp.Connection(host_and_ports=hosts)
conn.set_listener('', MyListener())
conn.start()
conn.connect('admin', 'admin', wait=True,headers = {'client-id': 'clientname'} )
conn.subscribe(destination='A.B.C.D', id=1, ack='auto',headers = {'subscription-type': 'MULTICAST','durable-subscription-name':'someValue'})

conn.send(body=' '.join(sys.argv[1:]), destination='A.B.C.D')

time.sleep(2)
conn.disconnect()

```

以下是此代码执行的任务:

*   为了从消息传递系统接收消息，我们需要在连接上设置一个侦听器，然后订阅目的地。
*   我们正在与本地端口 61616 上的代理建立连接。`Connection`的第一个参数是`host_and_ports`。这包含一个 IP 地址和消息代理监听 STOMP 连接的端口。
*   `start`方法创建一个到代理的套接字连接。
*   然后，我们使用带有凭证的`connect`方法来访问代理，并使用`headers` `client-id`来确保创建的订阅是持久的。
*   一旦使用`subscribe`方法建立了到代理的连接，我们就使用确认模式`auto`订阅目的地`A.B.C.D`。此外，我们必须将`headers`订阅类型作为`MULTICAST`提供，并将`durable-subscription-name`作为某个文本值提供。
*   要创建持久订阅，必须在`CONNECT`帧上设置`client-id`标头，并且必须在`SUBSCRIBE`帧上设置`durable-subscription-name`。这两个头的组合将形成持久订阅的标识。
*   在与代理建立连接之后，我们可以使用`send`方法向目的地 A.B.C.D .发送/生成消息。这里第一个参数是从命令行接受文本/字符串值，第二个参数是目的地名称或主题名称。

## **如何执行 Python 客户端**

*   确保 Apache ActiveMQ Artemis 代理配置为支持 STOMP 协议。默认情况下，端口 61616 被配置为支持几乎所有的邮件协议。

```
<acceptor name="artemis">tcp://0.0.0.0:61616?tcpSendBufferSize=1048576;tcpReceiveBufferSize=1048576;protocols=CORE,AMQP,STOMP,HORNETQ,MQTT,OPENWIRE;useEpoll=true;amqpCredits=1000;amqpLowCredits=300</acceptor>

```

*   要使用 STOMP 协议运行客户端，我们首先需要`stomp`模块，以便 STOMP API 的组件，如`connect`、`start`、`send`、`subscribe`和`disconnect`可用。所以先安装`stomp`模块。

```
pip install stomp.py

```

*   一旦安装了`stomp`模块，我们就可以通过以下方式轻松运行客户端:

```
[cpandey@vm254-231 python_stomp_example]$ python receiver_topic.py "Hello World"
received a message "Hello World"
[cpandey@vm254-231 python_stomp_example]$

```

*   我们可以从 Apache ActiveMQ Artemis 代理中使用以下命令来检查结果:

```
[cpandey@vm254-231 bin]$ ./artemis address show

A.B.C.D
DLQ

[cpandey@vm254-231 bin]$ ./artemis queue stat --user admin --password admin --url tcp://localhost:61616
OpenJDK 64-Bit Server VM warning: If the number of processors is expected to increase from one, then you should configure the number of parallel GC threads appropriately using -XX:ParallelGCThreads=N
|NAME |ADDRESS |CONSUMER_COUNT |MESSAGE_COUNT |MESSAGES_ADDED |DELIVERING_COUNT |MESSAGES_ACKED |
|DLQ |DLQ |0 |0 |0 |0 |0 |
|ExpiryQueue |ExpiryQueue |0 |0 |0 |0 |0 |
|clientname.someValue |A.B.C.D |0 |0 |1 |0 |1 |
[cpandey@vm254-231 bin]$

```

注意:A.B.C.D 是创建的`Address`，持久订阅是作为队列`clientname.someValue`创建的。

*   如果我们使用 Wireshark 读取网络转储，以下是完整的数据流:

```
STOMP
accept-version:1.1
client-id:clientname
login:admin
passcode:admin

.CONNECTED
version:1.1
session:4c98c896
server:ActiveMQ-Artemis/2.4.0.amq-711002-redhat-1 ActiveMQ Artemis Messaging Engine

.
SUBSCRIBE
ack:auto
destination:A.B.C.D
durable-subscription-name:someValue
id:1
subscription-type:MULTICAST

.SEND
content-length:4
destination:A.B.C.D

abcd.MESSAGE
subscription:1
content-length:4
message-id:30
destination:A.B.C.D
expires:0
redelivered:false
priority:4
persistent:false
timestamp:1528858440363

abcd.
DISCONNECT
receipt:6a8bc1fd-0c8b-4e13-871f-fbc9c8c4df9d

.RECEIPT
receipt-id:6a8bc1fd-0c8b-4e13-871f-fbc9c8c4df9d

```

就是这样。我希望这能帮助你对在 Apache ActiveMQ Artemis 或 Red Hat AMQ 7 中使用 STOMP 协议有一个基本的了解。

*Last updated: September 3, 2019*