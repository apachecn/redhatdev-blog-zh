# 使用 Nagios 被动检查进行警报管理器看门狗监控

> 原文：<https://developers.redhat.com/blog/2020/04/29/alertmanager-watchdog-monitoring-with-nagios-passive-checks>

安装一个新的[红帽 OpenShift](https://developers.redhat.com/openshift/) 集群后，进入**监控** - > **警戒**。在那里，您将发现一个监视警报，它发送消息让您知道 Alertmanager 不仅仍在运行，而且还发出您可能感兴趣的其他警报信号。您可以使用外部监控系统来挂钩看门狗警报，这反过来可以告诉您 OpenShift 集群中的警报正在工作。

你需要一张支票来检查你的支票是否有效

你是怎么做到的？在配置 Alertmanager 发送看门狗警报之前，我们需要接收端的一些东西，在我们的例子中是 Nagios。跟随我的旅程，通过被动检查获得 Alertmanager 的看门狗对 Nagios 的警报。

## 设置 Nagios

OpenShift 可能不是您管理下运行的第一个基础设施元素。这就是为什么我们开始用一个自制的(实际上是从 Python 3 网站和调整过的)Python HTTP 接收服务器从 OpenShift 中捕获一条消息，只是为了学习如何配置 alert manager 和可能修改接收到的 alert 消息。

此外，您可能已经有了 Nagios、Checkmk、Zabbix 或其他用于外部监控和运行警报的工具。对于这个旅程，我选择使用 Nagios，因为它是一个简单的通过`yum install nagios`预先烹饪和设置的选项。Nagios 通常只做*主动检查*。主动检查意味着 Nagios 是您配置的检查的发起者。为了知道 OpenShift Alertmanager 是否在工作，我们需要 Nagios 中的一个*被动* *检查*。

因此，让我们让我们已经存在的监视系统从 Alertmanager 接收一些东西。首先安装 Nagios 和所需的插件:

```
$ yum -y install nagios nagios-plugins-ping nagios-plugins-ssh nagios-plugins-http nagios-plugins-swap nagios-plugins-users nagios-plugins-load nagios-plugins-disk nagios-plugins-procs nagios-plugins-dummy
```

为了更加安全，我们使用`htpasswd`更改为 Nagios 管理员提供的默认密码:

```
$ htpasswd -b /etc/nagios/passwd nagiosadmin <very_secret_password_you_created>

```

**注意:**如果你也想把管理员的用户名`nagiosadmin`改成别的，别忘了也在`/etc/nagios/cgi.cfg`里改一下。

现在，我们可以第一次启用并启动 Nagios 了:

```
$ systemctl enable nagios
$ systemctl start nagios

```

不要忘记，每次修改配置文件时，都应该对它们进行健全性检查。在(重新)启动 Nagios Core 之前做这件事很重要，因为如果您的配置包含错误，它将不会启动。使用以下命令检查您的 Nagios 配置:

```
$ /sbin/nagios -v /etc/nagios/nagios.cfg
$ systemctl reload nagios
$ systemctl status -l nagios

```

## 将 HTTP POST 内容转储到文件中

在开始配置之前，我们首先需要一个 HTTP POST receiver 程序，以便通过 webhook 配置从 Alertmanager 接收消息。Alertmanager 向 HTTP 端点发送一条 JSON 消息。为此，我创建了一个非常基本的 python 程序，将通过 POST 接收的所有数据转储到一个文件中:

```
#!/usr/bin/env python3

from http.server import HTTPServer, BaseHTTPRequestHandler
from io import BytesIO

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):

def do_GET(self):
self.send_response(200)
self.end_headers()
self.wfile.write(b'Hello, world!')

def do_POST(self):
content_length = int(self.headers['Content-Length'])
body = self.rfile.read(content_length)
self.send_response(200)
self.end_headers()
response = BytesIO()
response.write(b'This is POST request. ')
response.write(b'Received: ')
response.write(body)
self.wfile.write(response.getvalue())
dump_json = open('/tmp/content.json','w')
dump_json.write(body.decode('utf-8'))
dump_json.close()

httpd = HTTPServer(('localhost', 8000), SimpleHTTPRequestHandler)
httpd.serve_forever()

```

上述程序肯定需要一些返工。对于 Nagios，文件中输出的位置和格式都必须更改。

## 为被动检查配置 Nagios

现在这个基本的接收程序已经就绪，让我们在 Nagios 中配置被动检查。我在文件`/etc/nagios/objects/commands.cfg`中添加了一个伪命令。这是我从 Nagios 文档中了解到的，但我并不清楚这是否是正确的位置和正确的信息。最后，这个过程对我起作用了。但是继续往下，最后的目的是 Alertmanager 出现在 Nagios 中。

将以下内容添加到`commands.cfg`文件的末尾:

```
define command {
command_name check_dummy
command_line $USER1$/check_dummy $ARG1$ $ARG2$
}

```

然后将其添加到服务器的服务对象`.cfg`文件中:

```
define service {
use generic-service
host_name box.example.com
service_description OCPALERTMANAGER
notifications_enabled 0
passive_checks_enabled 1
check_interval 15 ; 1.5 times watchdog alerting time
check_freshness 1
check_command check_dummy!2 "Alertmanager FAIL"
}

```

如果我们可以通过 curl 检查这是否正常工作，那就太好了，但是首先，我们必须更改示例 Python 程序。默认情况下，它写入一个文件，对于本例，它必须写入一个 Nagios `command_file`。

这是调整后的 Python 程序，用右边的`service_description`写入`command_file`:

```
#!/usr/bin/env python3

from http.server import HTTPServer, BaseHTTPRequestHandler
from io import BytesIO
import time;

class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):

def do_GET(self):
self.send_response(200)
self.end_headers()
self.wfile.write(b'Hello, world!')

def do_POST(self):
content_length = int(self.headers['Content-Length'])
body = self.rfile.read(content_length)
self.send_response(200)
self.end_headers()
response = BytesIO()
response.write(b'This is POST request. ')
response.write(b'Received: ')
response.write(body)
self.wfile.write(response.getvalue())
msg_string = "[{}] PROCESS_SERVICE_CHECK_RESULT;{};{};{};{}"
datetime = time.time()
hostname = "box.example.com"
servicedesc = "OCPALERTMANAGER"
severity = 0
comment = "OK - Alertmanager Watchdog\n"
cmdFile = open('/var/spool/nagios/cmd/nagios.cmd','w')
cmdFile.write(msg_string.format(datetime, hostname, servicedesc, severity, comment))
cmdFile.close()

httpd = HTTPServer(('localhost', 8000), SimpleHTTPRequestHandler)
httpd.serve_forever()

```

通过一点点`curl`，我们可以检查 Python 程序是否与`command_file`有连接，并且 Nagios 可以读取它:

```
$ curl localhost:8000 -d OK -X POST

```

现在我们只需触发 POST 操作。发送到 Nagios 的所有信息都是用这个 Python 程序硬编码的。对这种信息进行硬编码真的不是最佳实践，但它让我现在继续下去。此时，我们有了一个端点(`SimpleHTTPRequestHandler`)，可以通过 webhook 将 Alertmanager 连接到外部监控系统——在本例中，是带有 HTTP 助手程序的 Nagios。

## 在 Alertmanager 中配置 webhook

要配置警报管理器的看门狗，我们必须调整密码`alertmanager.yml`。要从 OpenShift 中获取该文件，请使用以下命令:

```
$ oc -n openshift-monitoring get secret alertmanager-main --template='{{ index .data "alertmanager.yaml" }}' |base64 -d > alertmanager.yaml

```

```
global:
  resolve_timeout: 5m
route:
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'default'
  routes:
  - match:
      alertname: 'Watchdog'
    repeat_interval: 5m
    receiver: 'watchdog'
receivers:
- name: 'default'
- name: 'watchdog'
  webhook_configs:
  - url: 'http://nagios.example.com:8000/'

```

**注意:**在普罗米修斯网页上，你可以看到[可能的警报端点](https://prometheus.io/docs/alerting/configuration/)。正如我在`webhook_config`中发现的，你应该在`alertmanager.yml`中用复数形式(`webhook_configs`)来命名那个文件。另外，查看 Prometheus GitHub 上提供的[示例。](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml)

要让我们的新配置回到 OpenShift 中，请执行以下命令:

```
$ oc -n openshift-monitoring create secret generic alertmanager-main --from-file=alertmanager.yaml --dry-run -o=yaml | oc -n openshift-monitoring replace secret --filename=-

```

最后，您将看到 Nagios 收到类似的内容。实际上，这是看门狗通过`webhook_config`发送给 Nagios 的消息:

```
{"receiver":"watchdog",
"status":"firing",
"alerts":[
{"status":"firing",
"labels":
{"alertname":"Watchdog",
"prometheus":"openshift-monitoring/k8s",
"severity":"none"},
"annotations":
{"message":"This is an alert meant to ensure that the entire alerting pipeline is functional.\nThis alert is always firing, therefore it should always be firing in Alertmanager\nand always fire against a receiver. There are integrations with various notification\nmechanisms that send a notification when this alert is not firing. For example the\n\"DeadMansSnitch\" integration in PagerDuty.\n"},
"startsAt":"2020-03-26T10:57:30.163677339Z",
"endsAt":"0001-01-01T00:00:00Z",
"generatorURL":"https://prometheus-k8s-openshift-monitoring.apps.box.example.com/graph?g0.expr=vector%281%29\u0026g0.tab=1",
"fingerprint":"e25963d69425c836"}],
"groupLabels":{},
"commonLabels":
{"alertname":"Watchdog",
"prometheus":"openshift-monitoring/k8s",
"severity":"none"},
"commonAnnotations":
{"message":"This is an alert meant to ensure that the entire alerting pipeline is functional.\nThis alert is always firing, therefore it should always be firing in Alertmanager\nand always fire against a receiver. There are integrations with various notification\nmechanisms that send a notification when this alert is not firing. For example the\n\"DeadMansSnitch\" integration in PagerDuty.\n"},
"externalURL":"https://alertmanager-main-openshift-monitoring.apps.box.example.com",
"version":"4",
"groupKey":"{}/{alertname=\"Watchdog\"}:{}"}

```

最后，如果一切顺利，您会在 Nagios 的服务概览中看到一个漂亮的绿色“OCPALERTMANEGER”服务
![](img/ec4ffa6c6da8ece89ea2d9b337e0b30a.png)

如果您想了解 Nagios 被动检查，请在 [Nagios 核心被动检查](https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/4/en/passivechecks.html)阅读更多内容。

感谢您加入我的旅程！

*Last updated: June 29, 2020*