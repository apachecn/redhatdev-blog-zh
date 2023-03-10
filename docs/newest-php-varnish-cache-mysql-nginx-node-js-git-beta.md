# 最新的 PHP、Varnish Cache、MySQL、NGINX、Node.js 和 Git 现在都在测试中

> 原文：<https://developers.redhat.com/blog/2018/10/24/newest-php-varnish-cache-mysql-nginx-node-js-git-beta>

我们很高兴地宣布立即提供[红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/) 3.2 测试版，它将这些组件添加到 [红帽企业版 Linux 7](https://developers.redhat.com/products/rhel/overview/) :

*   PHP 7.2
*   清漆缓存 6.0
*   MySQL 8.0 的实现
*   NGINX 1.14
*   Node.js 10
*   去 2.18 去
*   更新 Apache HTTP server 2.4

这些测试版在 Red Hat Enterprise Linux 7(dev tools 或 RHSCL channel)上可用于 x86_64、s390x、aarch64 和 ppc64le。在“新组件详细信息”一节中阅读关于每个组件的更多详细信息。

## ***关于红帽软件收藏***

Red Hat 每年两次发布新版本的编译器工具集、脚本语言、开源数据库和/或 web 工具，为应用程序开发人员提供最新、稳定的版本。这些红帽支持的产品被打包成 [红帽软件集合](https://developers.redhat.com/products/softwarecollections/overview/) (脚本语言、开源数据库、web 工具等。)、 [红帽开发者工具集](https://developers.redhat.com/products/developertoolset/overview/) (GCC)，以及最近新增的编译器工具集 [Clang/LLVM、Go、Rust](https://developers.redhat.com/products/clang-llvm-go-rust/overview/) 。所有这些都是可安装的，并且包含在所有 Red Hat Enterprise Linux 开发人员订阅和大多数 Red Hat Enterprise Linux 订阅中。大多数组件也可以作为 Linux 容器映像用于跨 Red Hat 平台的混合云开发，包括:Red Hat Enterprise Linux、Red Hat OpenShift、Red Hat OpenStack 等。

## ***新组件详情***

### **PHP 7.2**

这次 PHP 7.2 的增加标志着 PHP 7 系列的第二次特性更新，性能得到了显著的提高；它有许多改进和新功能:

*   转换对象/数组转换中的数字键
*   不可计数物体的计数
*   对象类型提示
*   HashContext 作为对象
*   将 TLS 常数提高到正常值

PHP 7.2 在 RHEL 7 上运行，所有架构。

包名:`rh-php72`

集装箱图片:`rhscl-beta/php-72-rhel7`

### **清漆缓存 6.0**

Varnish Cache 6.0 是一个 web 应用加速器，也称为缓存 HTTP 反向代理。它安装在一个讲 HTTP 的 web 服务器前面，配置为缓存内容，具有非常高的性能，具有高度可扩展的内置配置语言。清漆 6.0 新特性包括:

*   HTTP/2 支持——经过一段时间的测试，Varnish 6.0 现在完全支持 HTTP/2。
*   支持 Unix 域套接字(UDS ),包括客户端和后端服务器
*   清漆配置语言的新水平(VCL)，vcl 4.1
*   新的和改进的 Varnish 模块(vmod):vmod _ directors、vmod_proxy、vmod_unix、vmod_vtc

清漆缓存工作在 RHEL 7，所有架构。

包名:`rh-varnish6`

集装箱图片:`rhscl-beta/varnish-6-rhel7`

### **MySQL 8.0** 的实现

MySQL 8.0 提供了全面的改进，旨在使数据库管理员和开发人员能够在最新一代的开发框架和硬件平台上创建和部署下一代的 web、嵌入式、移动和云/SaaS/PaaS/DBaaS 应用。

MySQL 8.0 亮点包括:

*   交易数据字典
*   SQL 角色
*   默认为 utf8mb4
*   常用表表达式
*   窗口功能

MySQL 8.0 在 RHEL 7 上运行，所有架构。

包名:`rh-mysql80`

集装箱影像:`rhscl-beta/mysql-80-rhel7`

### **NGINX 1.14**

NGINX 1.14.0 是该项目的最新稳定版本，包括一个用于镜像请求的新镜像模块、HTTP/2 推送支持和限制并发推送请求的数量，以及一个用于将请求转发到 gRPC 服务器的 gRPC 代理模块。

NGINX 1.14 在 RHEL 7 上运行，所有架构。

包名:`rh-nginx114`

集装箱图片:`rhscl-beta/nginx-114-rhel7`

### **Node.js 10**

Node.js 是一个基于 JavaScript 运行时的现代编程平台，用于轻松构建快速、可扩展的网络应用。Node.js 使用事件驱动的非阻塞 I/O 模型，这使得它轻量级且高效，非常适合跨分布式设备运行的数据密集型实时应用程序。Node.js 10 版本中的其他特性包括:

*   增强的安全性。
*   N-API (Node.js API)，从 beta 版迁移到稳定版，提供独立于 V8 JavaScript 引擎底层 Node.js 变化的稳定模块 API，该 API 帮助模块维护人员和生产部署，使升级更容易。
*   JavaScript 语言改进，包括 prototype.toString()，它现在返回源代码文本的精确片段，以及对旁路漏洞的缓解，以防止信息泄漏。
*   错误处理的改进，采用错误代码来简化经常性的错误检查。
*   通过 V8 提升性能，包括异步发电机和阵列。
*   随着 Node.js 10 的发布，通过跟踪事件增加了对代码性能问题的可见性。
*   Node.js 10 版本中的 API 允许用户代码在运行时按需启用和禁用跟踪事件，以提高诊断应用程序问题的灵活性。

Node.js 10 在 RHEL 7 上工作，所有架构。

包名:`rh-nodejs10`

### **去 2.18** 去

Git 是一个开源的分布式版本控制系统，旨在快速高效地处理从小到大的项目。Git 包括廉价的本地分支、方便的暂存区和多个工作流等功能，这些功能是其他版本控制系统所不具备的。Git 允许并鼓励开发者拥有多个可以完全相互独立的本地分支。这些开发线的创建、合并和删除只需要几秒钟，这比其他源代码管理系统要快得多。Git 2.18 的特性有:

*   Git 2.18 中最重要的特性是引入了新的有线协议 v2，旨在提供更高的性能。这种新的协议被设计得更快，并且由于显著的性能优势已经被使用。
*   Git 大文件存储(LFS)在 Git 中用文本指针替换大文件，并将文件内容存储在远程服务器上。
*   Git 2.18 的其他变化主要是其他常规更新、错误修复和改进，包括各种其他性能优化。

Git 2.18 在 RHEL 7 上运行，所有架构。

包名:`rh-git218`

## ***该组件已在红帽软件集合 3.2*** 中更新

### **更新到 Apache HTTP Server 2.4**

Apache HTTP 是 Apache 软件基金会的一个项目，是互联网上排名第一的 HTTP 服务器。Apache HTTP Server 版的更新包括:

*   支持 OpenSSL 1.0.2 并包含 mod_md 模块。
*   对于现有的 Apache 2.2.x 用户来说，迁移到 2.4 非常容易，因为只有很少的配置更改。
*   使用 Apache 2.4，web 开发人员可以获得其他“快速”web 服务器的性能，而不必切换到更新的 web 服务器，如 Nginx。

Apache HTTP Server 2.4 可以在 RHEL 7、所有架构和 RHEL 6 上运行。

还是包装成`httpd24`。

集装箱图片:`rhscl-beta/httpd-24-rhel7`

## ***详情:***

*   [最快捷的方式你好世界](https://developers.redhat.com/products/softwarecollections/hello-world/) 使用红帽企业 Linux 。
***   **[红帽软件收藏](https://developers.redhat.com/products/softwarecollections/overview/) 。***   **在这里找到所有老版本的组件[](https://access.redhat.com/support/policy/updates/rhscl)。***   红帽软件集合[文档](https://access.redhat.com/documentation/en-us/red_hat_software_collections/)**

***Last updated: October 23, 2018***