# 将专用 NPM 模块用于 RHMAP

> 原文：<https://developers.redhat.com/blog/2018/02/05/private-npm-modules-rhmap>

在这篇博文中，我将尝试讲述如何将[红帽移动应用平台](https://access.redhat.com/products/red-hat-mobile-application-platform)与来自 registry.npmjs.org 的私有 npm 模块一起使用。

## NPM

### 私有国家预防机制模块

使用 npm 私有模块，您可以使用 npm 注册表来托管您自己的私有代码，并使用 npm 命令行来管理它。这使得将 Express 和 Browserify 等公共模块与您自己的私有代码一起使用变得很容易。

### 先决条件

*   私人套餐升级账户
*   Npm v 2.7.0 或更高版本

### 设置软件包

首先使用您的用户名登录:

`*npm login*`

如果包名以`@`开头，那么所有私有包都是作用域。范围是在`@`和斜线之间的一切。

`*@scope/project-name*`

个人用户的套餐应指定如下:

`*@username/project-name*`

发布包:

`npm publish`

### 设置客户端和云应用/MBaaS 服务

首先，我们需要更改 package.json 文件并添加新的私有模块:

```
{
  …
  "dependencies" : {
&nbsp;   "@username/project-name" : "1.0"
  }
}
```

当部署客户端或云应用程序时，服务器需要一种下载私有模块的方法。这可以使用`.npmrc`文件来解决。`.npmrc`文件将通过 npm 验证您的服务器。

Npm 使用身份验证令牌在 cli 中进行身份验证，以生成令牌:

`*npm login <username> <password>*`

这将在以下文件中生成令牌:

`*~/.npmrcs*`

带有以下信息:

`*//registry.npmjs.org/:_authToken=00000000-0000-0000-0000-000000000000*`

复制生成的`.npmrc`文件，粘贴到你的根项目(Cloud App/MBaaS 或 Mobile)文件夹。

将`.npmrc`添加到 git 存储库:

`*git add .npmrc*`

`*git commit -am “added .npmrc file”*`

`*git push*`

**注意:** 令牌并非源自您的密码，但更改密码会使所有令牌失效，令牌将一直有效，直到密码被更改。也可以通过注销计算机或从 npm 门户撤销令牌来使令牌失效。

**注 2:** 生成的令牌具有写/读权限，如果有人得到令牌，他们可能会做恶意的事情，为了防止这一点，我们还可以创建一个只读权限令牌并更新 `.npmrc` 文件:

`*npm token create --read-only*`

### 云应用/MBaaS 服务的附加步骤

工作室使用 fh-npm 下载包，这忽略了`.npmrc`文件，如果我们想强制工作室使用 npm，我们需要使用 shrinkwrap:

`*npm shrinkwrap*`

npm 封装允许您锁定 node_modules 目录中所有包年龄及其后代包年龄的版本号。它将生成一个 npm-shrinkwrap.json 文件。

`*git add npm-shrinkwrap.json*`

`*git commit -am “Added npm-shrinkwrap.json”*`

`*git push*`

**注意:** 该命令创建和更新的文件将优先于任何其他现有或未来的`package-lock.json`文件。

## 部署云 App / MBaaS 服务

部署云应用后，您将能够在控制台日志中看到私有模块是如何自动解析和下载的。

感谢达拉赫·考利提供写这篇文章所需的信息。

*Last updated: February 7, 2018*