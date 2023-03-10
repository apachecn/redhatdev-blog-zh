# 使用 Prometheus 监控 OpenShift 上的 Node.js 应用程序

> 原文：<https://developers.redhat.com/blog/2018/12/21/monitoring-node-js-applications-on-openshift-with-prometheus>

## 可观察性是关键

Node.js 最棒的一点是它在[容器](https://developers.redhat.com/blog/category/containers/)中的表现。其快速的启动时间和相对较小的尺寸使其成为 [OpenShift](https://www.openshift.com/) 上微服务应用的最爱。但是这种向容器化部署的转变带来了一些复杂性。因此，监控 Node.js 应用程序可能会很困难。有时，似乎我们的应用程序的性能和行为对我们来说变得不透明。那么，在问题变成问题之前，我们能做些什么来发现和解决我们服务中的问题呢？我们需要**通过监控我们服务的状态来增强可观察性**。

## 使用仪器

对我们的应用程序进行检测是增加可观察性的一种方式。因此，在本文中，我将使用 [Prometheus](https://prometheus.io/) 演示 Node.js 应用程序的插装。

Prometheus 是一个可安装的服务，它从您的应用程序中收集测量指标，并将它们存储为时序数据。对于在线服务，比如一个 [Express.js](https://expressjs.com) 应用程序，我们最关心的指标是吞吐量、错误和延迟。您的应用程序负责向 Prometheus 系统公开这些指标。因此，使用`prom-client` NPM 模块，我们将测试一个小型 Express.js 应用程序，并公开这些指标供 Prometheus 使用。

### 简单的 Express.js 应用程序

让我们从创建一个简单的 Express.js 应用程序开始。在这个应用程序中，我们在`/api/greeting`有一个服务端点，它将接受`GET`或`POST`请求，并返回一个问候作为`JSON`。以下命令将启动您的项目。

```
$ mkdir myapp
$ cd myapp
$ npm init -y
$ npm install --save express body-parser prom-client
```

这会为您创建一个`package.json`文件，并安装所有的应用程序依赖项。接下来，在文本编辑器中打开`package.json`文件，并将以下内容添加到`scripts`部分:`"start": "node myapp.js"`。

### 默认和自定义检测

`prom-client`模块公开了普罗米修斯自己推荐的所有[默认度量标准](https://prometheus.io/docs/instrumenting/writing_clientlibs/#standard-and-runtime-collectors)。按照链接阅读更多相关信息。默认值包括诸如`process_cpu_seconds_total`和`process_heap_bytes`之类的指标。除了公开这些默认度量之外，`prom-client`还允许开发人员定义他们自己的度量，如下面的代码所示。

### 应用程序源代码

应用程序代码是一个相当简单的 Express 应用程序。在文本编辑器中创建一个名为`myapp.js`的新文件，并将下面的代码粘贴到其中。

```
'use strict';
const express = require('express');
const bodyParser = require('body-parser');

// Use the prom-client module to expose our metrics to Prometheus
const client = require('prom-client');

// enable prom-client to expose default application metrics
const collectDefaultMetrics = client.collectDefaultMetrics;

// define a custom prefix string for application metrics
collectDefaultMetrics({ prefix: 'my_application:' });

// a custom histogram metric which represents the latency
// of each call to our API /api/greeting.
const histogram = new client.Histogram({
  name: 'my_application:hello_duration',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'status_code'],
  buckets: [0.1, 5, 15, 50, 100, 500]
});

// create the express application
const app = express();
const port = process.argv[2] || 8080;
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}));

// our API
app.use('/api/greeting', (request, response) => {
  // start the timer for our custom metric - this returns a function
  // called later to stop the timer
  const end = histogram.startTimer();
  const name = request.query.name ? request.query.name : 'World';
  response.send({content: `Hello, ${name}!`});
  // stop the timer
  end({ method: request.method, 'status_code': 200 });
});

// expose our metrics at the default URL for Prometheus
app.get('/metrics', (request, response) => {
  response.set('Content-Type', client.register.contentType);
  response.send(client.register.metrics());
});

app.listen(port, () => console.log(`Hello world app listening on port ${port}!`));
```

在上面的源文件中，我们在第 16 行创建了一个定制的`histogram`指标，我们用它来计算应用程序的延迟。接下来，在 API route `/api/greeting`中，我们在第 33 行启动度量的计时器作为第一个动作。然后，我们在完成第 37 行的请求后停止计时器。

### 安装应用程序

您可以通过运行以下命令在 OpenShift 中安装这个应用程序。

```
$ npx nodeshift --strictSSL=false --expose
```

这将创建构建、运行和向应用程序公开外部路由所需的所有必要的 OpenShift 对象。部署完成后，您可以浏览到新部署的应用程序。您可以在`/metrics`路径查看普罗米修斯指标，或者访问`/api/greeting`查看这个令人兴奋的 API 的运行情况！在命令行中，您可以使用以下命令获取新部署的应用程序的 URL。

```
$ oc get -o template route myapp --template="http://{{.spec.host}}/api/greeting"
```

如果一切正常，您将在浏览器中看到类似这样的内容:`{"content":"Hello, World!"}`。现在用这个命令获取应用程序公开的 Prometheus 指标的 URL。

```
$ oc get -o template route myapp --template="http://{{.spec.host}}/metrics"
```

## 安装普罗米修斯

OpenShift 附带了一个已经可用的 Prometheus 实例。但是，这个实例已经针对 Kubernetes 系统本身的工具进行了优化。因此，出于我们的目的，我们将在 OpenShift 项目中安装一个独立的 Prometheus 服务器，并将其指向我们的应用程序。

对我们来说幸运的是，OpenShift 的开发者已经提供了一些模板让 Prometheus 在 OpenShift 上的安装变得相对容易。

### 普罗米修斯配置文件

OpenShift Prometheus 模板依赖于存储为 Kubernetes 秘密的几个配置文件。因此，在安装 Prometheus 之前，我们需要确保我们的集群包含正确的安装配置文件。这些是`prometheus.yml`和`alertmanager.yml`。我们的看起来像这样。

**T2`prometheus.yml`**

```
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']

  # Scrape configuration for our hello world app
  - job_name: 'myapp'
    static_configs:
    - targets: ['myapp:8080']

```

**T2`alertmanager.yml`**

```
global:
# The root route on which each incoming alert enters.
route:
  # default route if none match
  receiver: alert-buffer-wh
receivers:
- name: alert-buffer-wh
  webhook_configs:
  - url: http://localhost:9099/topics/alerts

```

这大部分只是样板文件，但是如果你看看`prometheus.yml`的底部，你可以看到重要的部分。这是我们通知普罗米修斯我们的新应用`myapp`的地方。我们告诉普罗米修斯，它可以在`myapp`服务的 8080 端口上被发现。回想一下，我们在`/metrics`端点提供指标。这是普罗米修斯期望的默认值。

### 添加配置机密并部署

我们将使用 Kubernetes secrets 来存储这些文件，由模板创建的 Prometheus 实例将知道在哪里可以找到它们。在本地文件系统上创建配置文件之后，请确保登录到 OpenShift。然后键入以下内容，将文件本身和 Prometheus 系统添加到您的项目中。

```
# Create the prom secret
$ oc create secret generic prom --from-file=prometheus.yml

# Create the prom-alerts secret
$ oc create secret generic prom-alerts --from-file=alertmanager.yml

# Create the prometheus instance
$ oc process -f https://raw.githubusercontent.com/openshift/origin/master/examples/prometheus/prometheus-standalone.yaml | oc apply -f -
```

一旦 Prometheus 系统完全部署并启动，您就可以浏览 Prometheus 仪表板来查看一些指标！Prometheus 仪表板的 URL 显示在 OpenShift 控制台中。如果一切都已正确部署，您应该会看到类似如下的屏幕。

[![](img/a5b3eff86443388466b6450325fc9a08.png "OpenShift Console")](/sites/default/files/blog/2018/12/Screen-Shot-2018-12-13-at-10.07.12-AM.png)The OpenShift console displays deployments and external routes to your applications.The OpenShift console shows deployments and routes to your applications">

### 浏览普罗米修斯仪表板

如果您喜欢命令行，您可以键入`oc get -o template route prom --template="http://{{.spec.host}}"`来获得到 Prometheus 部署的路径。首次浏览 Prometheus 应用程序时，您需要登录。只需使用您用来登录控制台的 OpenShift 凭证。之后，点击`Status`菜单项，选择`Targets`。这将显示您的 Prometheus 实例被配置为收集哪些服务。如果你做的一切都是正确的，你会看到这样的屏幕。

[![](img/653e6e76b72e3df561d422c2e8db2625.png "Prometheus Targets")](/sites/default/files/blog/2018/12/Screen-Shot-2018-12-13-at-10.05.40-AM.png)Prometheus TargetsPrometheus Targets">

第一种配置是普罗米修斯自己刮！第二个配置是我们的应用程序`myapp`。

### 通过添加负载测试您的部署

接下来，让我们使用 [Apache `ab`](https://httpd.apache.org/docs/2.4/programs/ab.html) 在应用程序上生成一些负载，以便将一些数据放入 Prometheus。例如，在这里，我对 API 进行了 500，000 次访问，每次有 100 个并发请求。

```
$ ab -n 500000 -c 100 http://myapp-myproject.192.168.99.100.nip.io/api/greeting
```

生成负载后，我们可以返回到主 Prometheus 仪表板屏幕，并构造一个简单的查询来查看我们的服务在测试中的表现。我们将使用我们定制的`hello_duration`指标来测量延迟。在文本框中键入此查询。

```
 my_application:hello_duration_sum / my_application:hello_duration_count 
```

您可以试验 Prometheus 收集的其他指标，以探索对您的应用程序可能有意义的其他度量。例如，在上面的简单例子中，普罗米修斯提供了这个图表。

![Monitoring a Node.js application with Prometheus](img/d1d2022e31f94cd885193d69538508d6.png)

## 结论

正如您所看到的，检测您的服务所需的实际代码相对简单，并且不太冗长。但是，当我们开始检测我们的应用程序时，需要建立一些基础设施。此外，还必须考虑什么是与你的服务和环境最相关的信息。我鼓励您尝试一下本教程，并让我知道哪些查询对您有用！

*Last updated: September 3, 2019*