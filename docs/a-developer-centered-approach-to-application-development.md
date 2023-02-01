# 以开发人员为中心的应用程序开发方法

> 原文：<https://developers.redhat.com/blog/2020/07/03/a-developer-centered-approach-to-application-development>

你是否梦想有一个本地开发环境，它易于配置，并且独立于你目前*没有*工作的软件层而工作？我愿意。

作为一名软件工程师，我经历过启动不容易配置的项目的痛苦。当大部分技术文档已经过时，或者更糟的是，遗漏了许多步骤时，阅读技术文档是没有帮助的。我浪费了很多时间试图理解为什么我的本地开发环境不工作。

## 理想的情景

作为一名开发人员，在为项目做贡献之前，您必须满足一些先决条件。例如，您必须同意版本控制要求，并且您需要知道如何使用项目 IDE，如何使用包管理器，等等。

但仅此而已。你不需要仅仅为了满足一个想要重新发明轮子的建筑师的自负而学习一个缺乏文档记录的、自制的框架。您不需要运行外部虚拟机来模拟生产环境。作为开发人员，您可以自由地将时间投入到改进代码和增加产品价值上。

## 以开发人员为中心的应用程序开发方法

我写这篇文章的目的是描述构建一个以开发者体验为中心的 [Angular 8](https://angular.io/guide/build) 应用程序的策略。

应用的类型是附带的。我描述了一个客户端应用程序，但是我们可以将类似的技术应用于后端模块。在这种情况下，框架是有角的，但是我们可以对任何你喜欢的框架使用类似的技术。

**注:**简单介绍一下，Angular 是一个应用程序设计框架和开发平台，用于创建高效复杂的单页面应用程序。你可以在 [Angular 网站](https://angular.io/)了解更多。

示例应用程序是一个简单的 web 应用程序，带有身份验证，它对 REST 端点执行几个调用。我不会提供很多关于领域和业务逻辑的细节，因为这些因素与我的讨论无关。

这个用例的主要需求是增强开发人员的体验。战略由此而来。

**注意**:在我解决用例需求的策略直接涉及 Angular 和其他软件库的情况下，我会分享这些技术的细节。然而，我相信其他技术和框架也有类似的选择。

## 需求 1:客户端应用程序中没有后端信息

想象以下场景:客户端应用程序必须执行几个`GET`操作，这些操作将获取数据并显示在 web 页面上。您如何知道每个 REST 端点要调用的主机地址、协议和端口？

通常，我见过三种解决这个问题的方法:

*   在构建时将后端信息添加到应用程序中。
*   将后端信息作为参数传递给 web 应用程序，或者从环境变量中检索它。
*   在同一台机器上找到 web 应用程序和 REST 服务。这种方法让 web 应用程序在特定的端口和路径调用`localhost`。在这种情况下，我们“只”需要硬编码端口和协议。

不幸的是，在开发 web 应用程序时，这些策略都会导致黑洞:

*   您需要在调试时修改运行时状态。
*   您需要破解应用程序来模拟预期的启动。
*   最糟糕的是，你需要指向一个真正的共享开发或测试环境。

### 策略:反向代理

一个*反向代理*的概念非常简单。首先，我们把它看作是一个黑盒特性。

假设有人配置了托管你的 web 应用的机器，这样当你在一个特定的路径(例如，`/api`)上(通过`localhost`)呼叫自己时，每个呼叫都会被自动转发到 [API](https://developers.redhat.com/topics/api-management/) 服务器。在这种配置下，使用哪个地址、协议或端口并不重要。

**注意:**如果你想深入了解这个黑盒，你可以在 [Apache HTTPD](https://www.digitalocean.com/community/tutorials/how-to-use-apache-as-a-reverse-proxy-with-mod_proxy-on-ubuntu-16-04) 或 [NGINX](https://linuxize.com/post/nginx-reverse-proxy/) 上了解更多关于配置反向代理的信息。

### 反向代理换角

现在让我们考虑 Angular 中的一个反向代理，使用一个稍微不同的场景。假设您的静态文件由 Webpack dev 服务器在端口 4200 上提供服务，而一个 [Node.js](https://developers.redhat.com/blog/category/node-js/) 应用程序在端口 3000 上提供 API。图 1 展示了这个架构的流程(归功于 https://justirr . com/blog/2016/11/configure-proxy-API-angular-CLI/。)

您可以轻松地将全局变量`PROXY_CONFIG`配置为 Webpack 开发服务器生命周期的一部分。根据您的`angular.json`配置文件，您可以选择使用`proxy.conf.json`或`proxy.conf.js`。下面是一个`PROXY_CONFIG`文件的例子:

```
const PROXY_CONFIG = {
  "/api": {
    "target": "http://localhost:3000/",
    "secure": false,
    "logLevel": "debug",
    "changeOrigin": true
  }
};

module.exports = PROXY_CONFIG;

```

注意每个 HTTP 调用都必须指向`/api`。不需要指定任何其他信息。反向代理为我们完成剩下的工作，就像这样:

```
getPosts(): Observable {
  return this.http.get('/api/posts/');
}

```

只要您订阅了`getPosts()`，它就会调用目标地址(本例中为 http://localhost:3000/posts)。

**注意**:了解有关设置 Angular CLI 反向代理或 [Webpack dev server 反向代理的更多信息](https://webpack.js.org/configuration/dev-server/#devserver-proxy)。

## 要求 2:离线编码(没有互联网连接的编码)

编码时，您希望对外部世界的依赖性尽可能小。避免连接到共享的远程开发机器有很多原因。远程机器可能是:

*   最近没有更新。
*   缓慢，因为它的负荷。
*   延迟了，因为有 VPN。
*   不可用，因为有人正在更新它。
*   无法连接，因为您的互联网连接不工作。

然而，你*也*不想在本地启动开发机器的真实实例。这种情况可能:

*   具有难以模仿的第三方依赖性。
*   运行起来很重，例如，最低要求 32GB 的内存。
*   连接到数据库，在这种情况下，您必须要么安装数据库，要么连接到一个真正的远程实例。
*   很难更新，因为您的数据是历史数据，所以今天有效的数据明天可能就无效了。

### 策略:嘲弄数据

有几种解决方案可以让开发变得快速和敏捷。例如，您可以使用[容器](https://developers.redhat.com/topics/containers/)来提供隔离的和可复制的计算环境。

当开发一个 web 应用程序时，我相信使用模仿的 API 是有意义的。如果您正在使用 REST 端点，我推荐使用`[`json-server`](https://github.com/typicode/json-server)` 包，您可以在全局和本地安装它。如果你全球安装`json-server`，你可以在任何你喜欢的地方启动它。如果您在本地安装它，您可以将其作为开发环境的依赖项安装，然后创建一个节点包管理器(`npm`)脚本来启动一个定制的模拟服务器。

设置非常直观。假设你有一个 JSON 文件作为数据源；说吧，`db.json`:

```
db.json:
{
  "posts": [
    { "id": 1, "title": "json-server", "author": "typicode" }
  ],
  "comments": [
    { "id": 1, "body": "some comment", "postId": 1 }
  ],
  "profile": { "name": "typicode" }
}

```

您可以通过命令行启动该文件:

```
$ json-server --watch db.json
```

默认情况下，它从端口 3000 的`localhost`开始，因此如果您`GET http://localhost:3000/posts/1`，您将收到以下响应:

```
{ "id": 1, "title": "json-server", "author": "typicode" }

```

`GET`只是一个例子，您也可以使用其他 HTTP 动词。您也可以选择将编辑内容保存在原始文件中，或者保持不变。公开的 API 遵循 REST 标准，您可以对远程模式进行排序、过滤、分页和加载。

正如我前面提到的，您可以创建自己的脚本并以编程方式运行一个`json-server`实例:

```
const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router('db.json')
const middlewares = jsonServer.defaults()

server.use(middlewares)
server.use(router)
server.listen(3000, () => {
  console.log('JSON Server is running')
})

```

### 角度模拟数据

我可以建议几个策略，让你的 Angular 应用程序处理模拟数据。两者都是基于代理的。

**策略一**:配置[反向代理](#reverse-proxy)，指向目标中的`http://localhost:3000/`，这样每次调用都指向`json-server`实例。

**策略 2** :向代理添加一个定制的嘲讽规则，这样它就可以使用`bypass`参数返回特定路径的数据:

```
const PROXY_CONFIG = {
  '/api': {
    'target': 'http://localhost:5000',
    'bypass': function (req, res, proxyOptions) {
      switch (req.url) {
        case '/api/json1':
          const objectToReturn1 = {
            value1: 1,
            value2: 'value2',
            value3: 'value3'
          };
          res.end(JSON.stringify(objectToReturn1));
          return true;
        case '/api/json2':
          const objectToReturn2 = {
            value1: 2,
            value2: 'value3',
            value3: 'value4'
          };
          res.end(JSON.stringify(objectToReturn2));
          return true;
      }
    }
  }
}

module.exports = PROXY_CONFIG;

```

## 需求 3:开发代码不应该影响生产代码，反之亦然

你见过多少次这样的事情:

```
if (devMode) {...} else {...}
```

这段代码是我们称之为*代码味道*的一个例子，这意味着它将用于开发目的的代码与仅用于生产的代码混合在一起。面向生产的构建不应该包含与开发相关的代码，反之亦然。解决代码味道的方法是对不同的目标使用不同的构建。

代码味道出现在许多不同种类的用例中。例如，您的应用程序可以托管在单点登录(SSO)身份验证系统之后。用户第一次在浏览器中请求应用程序时，请求会被重定向到外部页面，要求提供凭据。

当您处于开发模式时，您不想处理重定向。不太复杂的认证服务是受欢迎的。

### 策略:使用文件替换策略

在 Angular 中，基于当前配置，可以指定文件替换策略。您可以很容易地使用此功能将用于开发目的的简单身份验证服务替换为生产所需的更强大、更复杂的身份验证服务:

```
"configurations": {
  "production": {
    "fileReplacements": [
      {
        "replace": "src/app/core/services/authenticator.ts",
        "with": "src/app/core/services/authenticator.prod.ts"
      }
    ],
    ...
  ...
}

```

代码库现在有两个独立的身份验证服务，它们被配置用于两种不同的环境。最重要的是，根据特定的构建参数，最终的工件中将只包含一个服务:

```
$ npm run ng build -c production
```

## 需求 4:了解当前在生产环境中运行的应用程序的版本

您是否始终知道在给定主机上运行的应用程序的版本？您可以使用构建参数(如构建时间或上次提交标识符)来确定您的当前环境是否针对最近的更新或错误修复进行了更新。

### 策略:使用`angular-build-info`

Angular 包括一个名为 [`angular-build-info`](https://www.npmjs.com/package/angular-build-info) 的命令行工具，可以在 Angular 项目的`src/`文件夹中生成一个`build.ts`文件。使用该工具，您可以将`build.ts`文件导入到您的 Angular 应用程序中，并使用导出的`buildInfo`变量:

```
import { Component } from '@angular/core';
import { environment } from '../environments/environment';
import { buildInfo } from '../build';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  constructor() {
    console.log(
      `\nBuild Info:\n` +
      ` ❯ Environment: ${environment.production ? 'production ?' : 'development ?'}\n` +
      ` ❯ Build Version: ${buildInfo.version}\n` +
      ` ❯ Build Timestamp: ${buildInfo.timestamp}\n`
    );
  }
}

```

请注意，`build.ts`内容必须进行版本控制，因此您需要在构建时执行以下脚本:

```
$ angular-build-info --no-message --no-user --no-hash

```

这些参数是可选的，以便您可以定制生成的`buildInfo`。

## 要求 5:管道中快速有效的质量检查

不管您是在本地启动一个构建管道，还是发送了一个 pull 请求，对整个项目质量有一个总体的了解都是非常好的。

### 策略:带有质量门的静态代码分析

当您需要度量软件的质量时，静态代码分析可能会有所帮助。它提供了几个关于可读性、可维护性、安全性等指标。而不实际执行软件本身。

如果您能够测量质量度量，那么您可以配置正式的修订，这可能有助于评估用于开发和发布软件新部分的过程。这种正式的修订被命名为*质量关*。

静态代码分析必须快速，结果清晰。您不希望滚动浏览冗余记录结果的页面。这很重要——阶段和顺序，你把质量关放在哪里。

对于这个需求，我会在测试执行之前和编译或传输之后立即设置质量关(假设正在发生)。我推荐这种布局有两个原因:

1.  如果静态代码不能编译或转换，它可以避免浪费时间检查静态代码。
2.  它避免了浪费时间对不满足团队定义的最低需求的代码执行一整套测试。

重要的是要记住，管道执行需要资源。一个好的开发人员不应该在没有首先执行本地质量检查的情况下就提交。您还可以通过缓存结果，或者仅对更改列表中涉及的文件执行静态代码分析，来减少要检查的文件数量。

## 结论

当你开始一个新项目时，非技术需求不应该降低你的生产力曲线。

作为一名开发人员，你不应该把时间浪费在配置问题上，或者一台有时工作有时不工作的开发机器上。预先处理好这些问题。快乐的开发人员花在编码上的时间比解决技术障碍的时间多。

改善你的开发者体验不是一次性的过程，而是一个渐进的过程。自动化总是有空间的。总有改进的余地。

*Last updated: June 13, 2022*