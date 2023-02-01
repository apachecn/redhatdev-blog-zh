# 为 JavaScript 前端构建无根容器

> 原文：<https://developers.redhat.com/blog/2021/03/04/building-rootless-containers-for-javascript-front-ends>

默认情况下，大多数[容器](https://developers.redhat.com/topics/containers)是作为根用户运行的。以 root 用户身份运行时，在受限端口上安装依赖项、编辑文件和运行进程要容易得多。然而，正如计算机科学中的通常情况一样，简单是有代价的。在这种情况下，以根用户身份运行的容器更容易受到恶意代码和攻击。为了避免那些潜在的[安全](https://developers.redhat.com/topics/security)漏洞， [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 不会让你作为根用户运行容器。这种限制增加了一层安全性并隔离了容器。

本文向您展示了如何在无根容器中运行 JavaScript 前端应用程序。这个例子建立在我上一篇文章 [*的代码之上，使得环境变量可以在前端容器*](https://developers.redhat.com/blog/2021/03/04/making-environment-variables-accessible-in-front-end-containers) 中访问。

## 构建无根容器

这是我们将在示例中使用的 docker 文件。正如我在[上一篇文章](https://developers.redhat.com/blog/2021/03/04/making-environment-variables-accessible-in-front-end-containers/)中所展示的，您可以使用这个 docker 文件从 Angular、React 或 Vue.js 应用程序中访问环境变量:

```
FROM node:14

ENV JQ_VERSION=1.6
RUN wget --no-check-certificate https://github.com/stedolan/jq/releases/download/jq-${JQ_VERSION}/jq-linux64 -O /tmp/jq-linux64
RUN cp /tmp/jq-linux64 /usr/bin/jq
RUN chmod +x /usr/bin/jq

WORKDIR /app
COPY . .
RUN jq 'to_entries | map_values({ (.key) : ("$" + .key) }) | reduce .[] as $item ({}; . + $item)' ./src/config.json | ./src/config.tmp.json && mv ./src/config.tmp.json config.json
RUN npm install && npm run build

FROM nginx:1.17
# Angular: ENV JSFOLDER=/usr/share/nginx/html/*.js
# React: ENV JSFOLDER=/usr/share/nginx/html/static/js/*.js
# VueJS: ENV JSFOLDER=/usr/share/nginx/html/js/*.js
COPY ./start-nginx.sh /usr/bin/start-nginx.sh
RUN chmod +x /usr/bin/start-nginx.sh
WORKDIR /usr/share/nginx/html
# Angular: COPY --from=0 /app/dist/ .
# React: COPY --from=0 /app/build .
# VueJS: COPY --from=0 /app/dist .
ENTRYPOINT [ "start-nginx.sh" ]

```

该容器使用两个阶段来构建最终容器。在第一阶段，它使用作为 root 运行的`node:14`映像。构建过程最终会丢弃这个容器，所以不需要担心。

第二级容器是需要保护的容器。`nginx`基本映像当前以 root 身份运行，主要是为了能够在端口 80 上运行，这需要特权访问才能启用。一旦这个容器准备好无根运行，它将在端口 8080 上运行。您需要更改容器的默认`nginx`配置，以便无根运行。您还需要确保服务器本身以非特权用户的身份运行。最后，用户需要访问几个文件和文件夹。

让我们从使这个容器成为无根容器开始。

## 创建 NGINX 配置文件

第一步是为 NGINX 创建新的配置文件。您可以从运行 NGINX 所需的最基本的配置文件开始，并在此基础上构建它:

```
worker_processes auto;
events {
  worker_connections 1024;
}
http {
  include /etc/nginx/mime.types;
  server {
    server_name _;
    index index.html;
    location / {
      try_files $uri /index.html;
      }
    }
}
```

接下来，您需要将服务器设置更改为在端口 8080 上运行，而不是默认的端口 80。您还需要更改 NGINX 用来提供文件的默认路径:

```
http {
  ...
  server {
    listen 8080;
    ...
    location / {
      root /code;
      ...
    }
  }
}
```

最终的`nginx.conf`文件应该是这样的:

```
worker_processes auto;
events {
  worker_connections 1024;
}
http {
  include /etc/nginx/mime.types;
  server {
    listen 8080;
    server_name _;
    index index.html;
    location / {
      root /opt/app;
      try_files $uri /index.html;
    }
  }
}
```

## 编辑 Dockerfile 文件

现在您已经有了一个新的 NGINX 配置文件，可以让服务器以普通用户的身份运行，是时候编辑 docker 文件了。这个修改后的容器将以用户`nginx`的身份运行。在这种情况下，NGINX 基本映像为非 root 用户提供。

在构建的第二步中，在用`FROM`语句指定了基本映像之后，可以复制新的 NGINX 配置文件来覆盖默认文件。然后，创建一个`/opt/app`文件夹并更改其所有权:

```
FROM nginx:1.17
COPY ./nginx.conf /etc/nginx/nginx.conf
RUN mkdir -p /opt/app && chown -R nginx:nginx /opt/app && chmod -R 775 /opt/app

```

不要忘记改变`JSFOLDER`变量。这将确保 bash 脚本仍然注入您的环境变量。

```
# Angular
# ENV JSFOLDER=/opt/app/*.js
# React
# ENV JSFOLDER=/opt/app/static/js/*.js
# VueJS
# ENV JSFOLDER=/opt/app/js/*.js

```

### 更改文件所有权

接下来，您需要授予 NGINX 运行一系列文件和文件夹的权限，以便进行缓存和日志记录。您可以在一条`RUN`语句中更改所有这些命令的所有权，使用&符号来链接命令:

```
RUN chown -R nginx:nginx /var/cache/nginx && \
   chown -R nginx:nginx /var/log/nginx && \
   chown -R nginx:nginx /etc/nginx/conf.d

```

NGINX 还需要一个`nginx.pid`文件。这个文件还不存在，所以您需要创建它并将所有权分配给`nginx`用户:

```
RUN touch /var/run/nginx.pid && \
   chown -R nginx:nginx /var/run/nginx.pid

```

### 更新组和权限

最后，您将更改这些文件和文件夹的组，并更改权限，以便 NGINX 可以读写这些文件夹:

```
RUN chgrp -R root /var/cache/nginx /var/run /var/log/nginx /var/run/nginx.pid && \
   chmod -R 775 /var/cache/nginx /var/run /var/log/nginx /var/run/nginx.pid

```

### 切换到无根用户

现在您已经调整了所有的权限，您可以使用`USER`语句告诉 Docker 切换到`nginx`用户。然后，您可以使用`--chown`标志将文件从构建器步骤复制到`/opt/app`文件夹中，这使得`nginx`用户可以访问这些文件。最后，您将告诉 Docker 这个新映像使用了一个不同的端口。将`EXPOSE`语句用于端口 8080:

```
USER nginx
WORKDIR /opt/app
COPY --from=builder --chown=nginx  .
RUN chmod -R a+rw /opt/app
EXPOSE 8080

```

最终的前端 docker 文件将如下所示:

```
FROM node:14

ENV JQ_VERSION=1.6
RUN wget --no-check-certificate https://github.com/stedolan/jq/releases/download/jq-${JQ_VERSION}/jq-linux64 -O /tmp/jq-linux64
RUN cp /tmp/jq-linux64 /usr/bin/jq
RUN chmod +x /usr/bin/jq

WORKDIR /app
COPY . .
RUN jq 'to_entries | map_values({ (.key) : ("$" + .key) }) | reduce .[] as $item ({}; . + $item)' ./src/config.json | ./src/config.tmp.json && mv ./src/config.tmp.json config.json
RUN npm install && npm run build

FROM nginx:1.17
# Angular
# ENV JSFOLDER=/opt/app/*.js
# React
# ENV JSFOLDER=/opt/app/static/js/*.js
# VueJS
# ENV JSFOLDER=/opt/app/js/*.js
COPY ./nginx.conf /etc/nginx/nginx.conf
RUN mkdir -p /opt/app && chown -R nginx:nginx /opt/app && chmod -R 775 /opt/app
RUN chown -R nginx:nginx /var/cache/nginx && \
   chown -R nginx:nginx /var/log/nginx && \
   chown -R nginx:nginx /etc/nginx/conf.d
RUN touch /var/run/nginx.pid && \
   chown -R nginx:nginx /var/run/nginx.pid
RUN chgrp -R root /var/cache/nginx /var/run /var/log/nginx /var/run/nginx.pid && \
   chmod -R 775 /var/cache/nginx /var/run /var/log/nginx /var/run/nginx.pid
COPY ./start-nginx.sh /usr/bin/start-nginx.sh
RUN chmod +x /usr/bin/start-nginx.sh

EXPOSE 8080
WORKDIR /opt/app
# Angular
# COPY --from=0 --chown=nginx /app/dist/ .
# React
# COPY --from=0 /app/build .
# VueJS
# COPY --from=0 /app/dist .
RUN chmod -R a+rw /opt/app
USER nginx
ENTRYPOINT [ "start-nginx.sh" ]
```

您的新 docker 文件已准备就绪！您可以通过使用一个`docker build`后跟一个`docker run`来测试它。不要忘记映射新端口，因为这个容器不再在端口 80 上运行:

```
docker build -t frontend .
docker run -d -p 8080:8080 --rm --name front -e ENV=prod -e BASE_URL=/api frontend

```

## 结论

现在，您已经拥有了在安全容器中运行 JavaScript 前端所需的一切。无论您使用 Angular、React 还是 Vue.js，都可以在您的所有 JavaScript 项目中重用我们在本文中开发的映像。前端不仅可以安全运行，还可以让您将环境变量注入到代码中。你可以在 [GitHub](http://github.com/joellord/frontend-containers) 上找到这篇文章的所有例子和源代码。