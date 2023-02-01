# 使环境变量在前端容器中可访问

> 原文：<https://developers.redhat.com/blog/2021/03/04/making-environment-variables-accessible-in-front-end-containers>

当[使用任何现代](https://developers.redhat.com/topics/containers) [JavaScript 框架](https://developers.redhat.com/topics/javascript)(如 Angular、React 或 Vue.js)为单页面应用构建容器时，您可能会发现配置设置会根据容器运行的位置而有所不同。一个典型的例子是 API 的基本 URL，它会根据您是测试应用程序还是将其部署到生产环境中而有所不同。开发人员通常使用环境变量来解决这个问题。

环境变量通常在后端工作，因为那是代码运行的地方。但是如果你的应用程序存在于用户的浏览器中呢？有很多方法可以绕过这个限制。在某些情况下，您可能会构建一个端点保存必要参数的服务器。另一个解决方法是使用 PHP 在 JavaScript 代码中注入环境变量作为全局变量。这两个选项都可以，但是最好将环境变量作为容器构建过程的一部分注入。这样，您就不必更改代码库，并且仍然可以使用像 NGINX 这样的静态 web 服务器来交付应用程序内容。

本文向您展示了如何在构建容器时将环境变量直接注入到代码库中。

## 生产构建中的 JavaScript 框架

使用哪种 JavaScript 框架并不重要——React、Angular 或 vue . js——因为它们的工作方式几乎都是一样的。该框架运行一个监视文件的服务器，并在检测到更改时刷新浏览器。这个过程非常适合开发目的，但不太适合生产服务器。所有这些代码都需要太多的资源来运行。为了让应用程序内容能够在 web 服务器上工作，我们需要一个构建步骤来最小化代码并只保留必要的部分。然后，我们可以使用包含所有应用程序的 HTML、JavaScript 和 CSS 的单个页面创建一个包。当容器在生产环境中运行时，它将服务于这个缩小的包。

事实证明，为生产准备代码的容器构建步骤也是注入环境变量的好地方。我们将在接下来的部分中介绍这个过程。

## 创建框架应用程序

让我们从为您的 JavaScript 框架使用命令行界面(CLI)构建的框架应用程序开始:

```
# Angular
npx @angular/cli new angular-project
# React
npx create-react-app react-project
# VueJS
npx @vue/cli create vue-project

```

对于您选择的项目，在`/src`文件夹中创建一个`config.json`文件。该文件将包含可能根据环境而改变的设置。在这种情况下，它将有两个属性:一个用于指定环境，另一个用于虚拟 API 的基本 URL:

```
{
  "ENV": "development",
  "BASE_URL": "http://localhost:3000"
}
```

为简单起见，您使用的应用程序将在主页上显示这些值。转到您的主页，导入配置文件，并在视图中显示这两个值。

接下来，我们将查看 Angular、React 和 Vue.js 的应用程序特定代码。

### 有角的

要导入一个 JSON 文件，您可能需要将以下选项添加到您的`tsconfig.json`文件的`compilerOptions`中:

```
   "resolveJsonModule": true,
   "esModuleInterop": true,
   "allowSyntheticDefaultImports": true,

```

以下是应用程序组件(`src/app/app.component.ts`):

```
import { Component } from '@angular/core';
import Config from "../config.json";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  environment = Config.ENV;
  baseUrl = Config.BASE_URL;
}
```

下面是应用程序 HTML ( `src/app/app.component.html`):

```
<div>
  <p>Environment: {{ environment }}</p>
  <p>Base Url: {{ baseUrl }}</p>
</div>

```

### 反应

下面是 React ( `src/App.js`)的应用程序配置:

```
import Config from "./config.json";

function App() {
  const environment = Config.ENV;
  const baseUrl = Config.BASE_URL;
  return (
    <div>
      <p>Environment: { environment }</p>
      <p>Base Url: { baseUrl }</p>
    </div>
  );
}

export default App;
```

### view . js-检视. js

下面是 Vue.js ( `src/App.vue`)的配置:

```
<template>
  <div>
    <p>Environment: {{ environment }}</p>
    <p>Base Url: {{ baseUrl }}</p>
  </div>
</template>

<script>
import Config from "./config.json";

export default {
  name: 'App',
  data: () => {
    return {
      environment: Config.ENV,
      baseUrl: Config.BASE_URL
    }
  }
}
</script>

```

## 多阶段构建容器

现在，您已经准备好构建前端容器了。对于这个过程，您将使用一个容器来创建应用程序的生产版本。Docker 然后将这个构建函数的输出复制到第二个容器中，一个 NGINX 服务器。一旦创建了第二个容器，就要丢弃第一个容器。剩下的是 NGINX 服务器，其中包含前一阶段的最少文件。

让我们从创建一个包含应用程序的图像开始。稍后，我们将回来应用环境变量。在此阶段，您将执行以下操作:

1.  创建一个名为`Dockerfile`的新文件。第一阶段使用一个`node:14`映像来构建应用程序的生产版本。将所有文件复制到容器中。
2.  复制文件，然后运行一个`npm install`来获取项目的依赖项，并运行一个`npm run build`来创建生产资产。
3.  用一个`FROM nginx:1.17`语句开始第二阶段，并将第一阶段的文件复制到这个新容器中。

**注意**:为了避免复制不必要的文件，如`node_modules`文件夹，在`Dockerfile`所在的文件夹中创建一个`.docker-ignore`文件，并列出要忽略的文件夹。另外，请注意，生产代码的位置根据您使用的 JavaScript 框架而有所不同，因此取消注释您需要的代码行。Angular 要求您手动更改项目的名称。

以下是此阶段的完整 Dockerfile 文件:

```
FROM node:14
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM nginx:1.17
WORKDIR /usr/share/nginx/html
# Angular
# COPY --from=0 /app/dist/<projectName> .
# React
# COPY --from=0 /app/build .
# VueJS
# COPY --from=0 /app/dist .

```

创建 docker 文件后，您可以构建映像并启动容器进行测试。运行以下命令并打开浏览器到 [http://localhost:8080](http://localhost:8080/) :

```
docker build -t front-end.
docker run -d -p 8080:80 --rm --name front frontend

```

要在测试后停止容器，请输入:

```
docker stop front
```

## 注入环境变量

接下来，您将编辑 docker 文件来注入您的环境变量。首先，您将覆盖原始`config.json`文件的内容，然后您将调整 NGINX 服务器来注入环境变量。

### 覆盖 config.json

每个属性的值不是实际值，而是“`$key`”。产生的`config.json`如下所示:

```
{
  ENV: "$ENV",
  BASE_URL: "$BASE_URL"
}

```

您将使用`envsubst`在服务器启动之前将`$KEY`值更改为环境变量的真实值。要做到这一点，您需要在 docker 文件的第一步添加指令，以包含 [jq](https://stedolan.github.io/jq/manual/) ，这是一个使从 CLI 编辑 JSON 文件内容变得容易的工具。就在 docker 文件中的`FROM`行之后，添加以下内容以将`jq`安装到容器中:

```
ENV JQ_VERSION=1.6
RUN wget --no-check-certificate https://github.com/stedolan/jq/releases/download/jq-${JQ_VERSION}/jq-linux64 -O /tmp/jq-linux64
RUN cp /tmp/jq-linux64 /usr/bin/jq
RUN chmod +x /usr/bin/jq

```

文件复制完成后，您可以使用`jq`编辑`config.json`:

```
RUN jq 'to_entries | map_values({ (.key) : ("$" + .key) }) | reduce .[] as $item ({}; . + $item)' ./src/config.json > ./src/config.tmp.json && mv ./src/config.tmp.json ./src/config.json

```

**注意**:如果您想了解更多关于本例中使用的`jq`过滤器的信息，并尝试其他选项，您可以在 [jqTerm](https://jqterm.com) 中运行它。

### 调整 NGINX 服务器

修改完`config.json`文件后，您将调整 NGINX 服务器来注入环境变量。为此，您需要在启动 NGINX 服务器之前创建一个要执行的脚本。

这个文件(`start-nginx.sh`)包含了相当多的 bash 脚本。脚本的第一行运行一个命令来获取所有现有环境变量的名称，并将它们存储在`$EXISTING_VARS`中。然后，该脚本遍历生产文件夹中的每个 JavaScript 文件，并用该环境变量的实际值替换任何`$VARIABLE`。完成后，它用默认命令启动 NGINX 服务器:

```
#!/usr/bin/env bash
export EXISTING_VARS=$(printenv | awk -F= '{print $1}' | sed 's/^/\$/g' | paste -sd,);
for file in $JSFOLDER;
do
  cat $file | envsubst $EXISTING_VARS | tee $file
done
nginx -g 'daemon off;'

```

**注意**:每个框架的 JavaScript 文件的位置不同。在 Dockerfile 文件中设置了`$JSFOLDER`变量，这样您就可以取消注释您需要的行。

现在，将这个文件添加到容器中，并用这个新脚本覆盖 NGINX 映像的默认入口点。就在第二阶段的`FROM`语句之后，为您的框架添加以下几行:

```
# Angular
# ENV JSFOLDER=/usr/share/nginx/html/*.js
# React
# ENV JSFOLDER=/usr/share/nginx/html/static/js/*.js
# VueJS
# ENV JSFOLDER=/usr/share/nginx/html/js/*.js
COPY ./start-nginx.sh /usr/bin/start-nginx.sh
RUN chmod +x /usr/bin/start-nginx.sh

```

在文件的最后，添加新的入口点:

```
ENTRYPOINT [ "start-nginx.sh" ]

```

您的最终 docker 文件应该看起来像这样。您可以取消对所需行的注释，并删除所有其他注释语句:

```
FROM node:14
ENV JQ_VERSION=1.6
RUN wget --no-check-certificate https://github.com/stedolan/jq/releases/download/jq-${JQ_VERSION}/jq-linux64 -O /tmp/jq-linux64
RUN cp /tmp/jq-linux64 /usr/bin/jq
RUN chmod +x /usr/bin/jq
WORKDIR /app
COPY . .
RUN jq 'to_entries | map_values({ (.key) : ("$" + .key) }) | reduce .[] as $item ({}; . + $item)' ./src/config.json > ./src/config.tmp.json && mv ./src/config.tmp.json ./src/config.json
RUN npm install && npm run build

FROM nginx:1.17
# Angular
# ENV JSFOLDER=/usr/share/nginx/html/*.js
# React
# ENV JSFOLDER=/usr/share/nginx/html/static/js/*.js
# VueJS
# ENV JSFOLDER=/usr/share/nginx/html/js/*.js
COPY ./start-nginx.sh /usr/bin/start-nginx.sh
RUN chmod +x /usr/bin/start-nginx.sh
WORKDIR /usr/share/nginx/html
# Angular
# COPY --from=0 /app/dist/<projectName> .
# React
# COPY --from=0 /app/build .
# VueJS
# COPY --from=0 /app/dist .
ENTRYPOINT [ "start-nginx.sh" ]

```

## 重建您的映像并启动服务器

现在，您已经准备好重新构建您的映像，并再次启动服务器，但是这一次使用环境变量。在 [http://localhost:8080](http://localhost:8080/) 打开您的浏览器，您应该看到应用程序使用您传递给 Docker:

```
docker build -t frontend .
docker run -d -p 8080:80 --rm --name front -e ENV=prod -e BASE_URL=/api frontend

```

## 结论

总之，下面是使您的环境变量在您的前端容器中可访问的步骤:

1.  在你的`/src`文件夹中添加一个 [config.json](https://github.com/joellord/frontend-containers/blob/main/config.json) 文件。
2.  将 [start-nginx.sh](https://github.com/joellord/frontend-containers/blob/main/start-nginx.sh) bash 脚本添加到项目中。
3.  使用完成的[docker 文件](https://github.com/joellord/frontend-containers/blob/main/Dockerfile)来构建您的项目。
4.  使用`-e`启动容器来指定环境变量。

按照这些步骤创建 docker 文件后，您可以在任何 JavaScript 项目中重用它。`config.json`中的所有变量都会自动改变，你不再需要考虑它们。您可以在 [GitHub](https://github.com/joellord/frontend-containers) 上找到本文中使用的 Angular、React 和 Vue.js 应用程序的完整源代码和示例。