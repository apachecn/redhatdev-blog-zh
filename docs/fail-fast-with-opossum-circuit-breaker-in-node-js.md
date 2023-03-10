# Node.js 中负鼠断路器的快速故障

> 原文：<https://developers.redhat.com/blog/2021/04/15/fail-fast-with-opossum-circuit-breaker-in-node-js>

[微服务](/topics/microservices)模式是当今软件架构的标准。微服务让你把应用程序分解成小块，避免一个巨大的整体。唯一的问题是，如果这些服务中的一个失败，它可能会对您的整个体系结构产生连锁反应。

幸运的是，有另一种模式可以帮助解决这个问题:断路器模式。

本文解释了什么是断路器，以及如何在您的 [Node.js](/topics/nodejs) 应用程序中使用该模式。我们将使用[负鼠](https://www.npmjs.com/package/opossum)，断路器模式的 Node.js 实现。

## 什么是断路器？

在我们深入一个例子之前，让我们快速定义一个断路器，以及如何在你的代码中使用这个模式。

如果你尝试过同时运行太多的家用电器，你可能已经对断路器很熟悉了。由于大量电流涌入，电灯熄灭了。要恢复电力，你需要去地下室，找到电箱，找到“跳闸”的断路器，并重置它。断路器通过在电涌时关闭来保护你的住所。

在处理通过网络通信的微服务时，断路器模式的工作方式类似。其目的是减少运行速度太慢或由于网络故障而无法到达的服务的影响。断路器监控这种故障。一旦故障达到特定阈值，电路就会“跳闸”，此后的任何调用要么返回错误，要么采用回退响应。然后，经过设定的时间后，断路器对受影响的服务进行测试调用。如果呼叫成功，电路关闭，交通再次开始流动。

当多种服务相互依赖时，断路器尤其重要。如果一项服务出现故障，可能会导致整个架构瘫痪。还记得*星球大战*电影系列中的第一次死星爆炸吗？一个好的断路器可能会避免这一点。

## 什么是负鼠？

负鼠是 Node.js 的断路器，当事情开始失败时，负鼠装死，失败得很快。如果您愿意，可以提供一个在失败状态下执行的回退功能。

自 2016 年末以来，负鼠一直是一个社区项目，现在每周下载量超过 7 万次。它得到了 [Nodeshift](https://nodeshift.dev) 社区的支持。最近，Red Hat 发布了一个完全受支持的负鼠版本，通过 Red Hat 的客户注册表作为`@redhat/opossum`分发。负鼠将永远是一个社区项目，但如果你想知道你正在使用的版本有红帽的支持，那么`@redhat/opossum`版本可能适合你。你可以在这里了解更多关于红帽的 Node.js 产品[。](https://access.redhat.com/articles/5641561)

以下部分展示了如何将该模块添加到应用程序中，以及如何使用它来保护您的微服务。

## 向您的应用程序添加红帽负鼠

将`@redhat/opossum`模块添加到您的应用程序就像添加任何其他 Node.js 模块一样，只有一个小的变化。因为您将从 Red Hat 客户注册中心下载这个模块，所以您需要告诉`npm`从 Red Hat 注册中心下载任何带有`@redhat`名称空间的模块，同时继续从上游 NPM 注册中心下载所有其他模块。

首先，在应用程序的根目录中添加一个`.npmrc`文件。该文件应该如下所示:

```
@redhat:registry=https://npm.registry.redhat.com
registry=https://registry.npmjs.org

```

有了这个文件，您可以成功运行以下命令:

```
$ npm install @redhat/opossum

```

为了在您的应用程序中插入模块，您可以为每个其他 Node.js 模块插入同样的语句:

```
const CircuitBreaker = require(‘@redhat/opossum’)

```

现在，让我们看一个例子。

## 示例:Node.js 的负鼠断路器

对于这个例子，我们将使用节点移位[断路器启动器应用](https://github.com/nodeshift-starters/nodejs-circuit-breaker-redhat)。

**注意**:这个例子在社区和红帽版本的负鼠上的工作方式是一样的。

该示例由两个简单的 Node.js 微服务组成，所以让我们来看看这两个服务。

### 问候服务

`greeting-service`是应用程序的入口点。一个简单的 web 页面调用`greeting` REST 端点。然后，这个端点通过断路器调用第二个服务。该网页还有一个按钮，可以让您打开或关闭名称服务(我稍后将介绍),以模拟网络故障。

下面是负责问候服务的代码:

```
...
// We require Opossum
const Opossum = require('@redhat/opossum');
…

// Set some circuit breaker options
const circuitOptions = {
  timeout: 3000, // If name service takes longer than .3 seconds, trigger a failure
  errorThresholdPercentage: 50, // When 50% of requests fail, trip the circuit
  resetTimeout: 10000 // After 10 seconds, try again.
};
…

// Use a circuit breaker for the name service and define fallback function
const circuit = new Opossum(nameService, circuitOptions);
circuit.fallback(_ => 'Fallback');

…

// Greeting API
app.get('/api/greeting', (request, response) => {
 // Using the Circuits fire method to execute the call to the name service
  circuit.fire(`${nameServiceHost}/api/name`).then(name => {
    response.send({ content: `Hello, ${name}`, time: new Date() });
  }).catch(console.error);
});

```

接下来，我们将`nameService`函数作为第一个参数传递给断路器。它看起来像下面这样，这是使用`axios`对另一个端点的标准调用:

```
'use strict';
const axios = require('axios');

module.exports = endpoint => {
  return new Promise((resolve, reject) => {
    axios.get(endpoint)
      .then(response => {
        if (response.status !== 200) {
          return reject(new Error(`Expected status code 200, instead got ${response.status}`));
        }

        resolve(response.data);
      })
      .catch(reject);
  });
};

```

### 名称服务

另一个微服务`name-service`是一个 REST 端点，它根据我之前提到的 on 或 off 切换发送回一个响应。

启动应用程序非常简单。从存储库的根目录中，运行`./start-localhost.sh`文件来启动两个 Node.js 进程。该脚本还会尝试在运行应用程序的位置打开一个 web 浏览器。

按下网页上的**调用**按钮接触第一个端点。端点发送回一个响应，说明它是否可以联系第二个服务或者必须使用回退。您可以单击切换按钮来模拟网络故障。

## 结论

本文展示了断路器如何帮助减少微服务中的意外故障。您可以使用`@redhat/opossum`模块将这个模式添加到 Node.js 应用程序中。要了解有关这一新支持产品的更多信息，请查看 Red Hat 客户门户网站上的文章 [*Opossum:完全支持 Red Hat 构建 Node.js*](https://access.redhat.com/articles/5890311) 的断路器模块。

请参阅这些参考资料，了解关于本文中讨论的主题的更多信息:

*   有关断路的更多信息，请参见微服务架构的[断路器模式介绍](https://microservices.io/patterns/reliability/circuit-breaker.html)。
*   另请参阅 Martin Fowler 关于断路器模式的非常好的[文章。](https://martinfowler.com/bliki/CircuitBreaker.html)
*   参见[负鼠 API 文档](https://access.redhat.com/webassets/avalon/d/red_hat_build_of_node/opossum/5.0.0/jsdoc/)了解更多关于你可以用`@redhat/opossum`做什么。
*   请访问 Node.js 登录页面，了解 Red Hat 还在 Node.js 上做了什么。

*Last updated: October 14, 2022*