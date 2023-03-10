# 在 Red Hat OpenShift 上开始使用 Node.js 14

> 原文：<https://developers.redhat.com/blog/2020/10/20/get-started-with-node-js-14-on-red-hat-openshift>

4 月，Node.js 开发团队发布了 [Node.js 14](https://nodejs.org/en/blog/release/v14.0.0/) 。这个代号为 Fermium 的主要版本将在 2020 年 10 月成为长期支持(LTS)版本。

![The Red Hat OpenShift logo.](img/da9d33d681050cf79386f12a783fa5b9.png)

Node.js 14 融合了 V8 8.1 JavaScript 引擎的改进和新特性。我将介绍其中的两个:可选链接和无效合并操作符。我还将向您展示如何在 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 上部署 Node.js 14。要了解更多 Node.js 14 中的改进和新特性，请参阅文章末尾的参考资料列表。

**注意**:[Red Hat Software Collections](https://developers.redhat.com/products/softwarecollections/overview)团队为 Node.js 创建并维护源到映像(Source-to-Image，S2I)容器映像。他们已经为 Node.js 14 发布了一个 S2I 映像。

## Node.js 14 中的可选链接

JavaScript 的可选链接操作符(`?.`)允许您读取位于连接对象链深处的属性值。有了这个特性，您不需要显式地验证链中的每个引用。下面是 Node.js 14 中可选链接的一个示例:

```
const adventurer = {
  name: 'Alice',
  cat: {
    name: 'Dinah'
  }
};

console.log(adventurer.dog?.name); // undefined (no error)

```

在以前的 Node.js 版本中，我们可能会使用逻辑 AND ( `&&`)运算符来解决这个问题，如下所示:

```
console.log(adventurer.dog && adventurer.dog.name);

```

如果`&&`运算符左边的操作数无效，那么右边的操作数将不会被计算。最终，JavaScript 将返回一个未定义的 T2，而不是一个错误。

## 看涨凝聚算子

Nullish 合并(`??`)是一个逻辑运算符，当左边的操作数为空或未定义时，返回右边的操作数。否则，它返回其左侧的操作数:

```
null ?? "n/a" // "n/a"

undefined ?? "n/a" // "n/a"

false ?? true // false

0 ?? 100 // 0

"" ?? "n/a" // ""

NaN ?? 0 // NaN

```

这段代码片段中显示的所有操作数都是 *falsy* 值，这意味着当它们被强制转换为布尔值时，计算结果为 false。如果我们使用更熟悉的逻辑 OR 运算符(`||`)，前面表达式的计算将会不同:

```
false || true // true

0 || 100 // 100

"" || "n/a" // "n/a"

NaN || 0 // 0

```

在为可空值提供后备值时，我们建议使用`??`而不是`||`。

## 在 OpenShift 上部署 Node.js 14 的两种方法

如果你熟悉使用 S2I 图像的过程，你已经知道该怎么做了。这个讨论是针对那些不熟悉使用 S2I 图像的开发人员的。

至少有两种方法可以使用新的 Node.js 14 映像快速部署应用程序。一种选择是将`oc new-app command`与 Git repo 一起使用:

```
oc new-app registry.access.redhat.com/rhel8-beta/nodejs-14:latest~https://github.com/nodeshift-starters/nodejs-rest-http

oc expose svc/nodejs-rest-http

```

或者，您可以使用 [Nodeshift 模块](https://www.npmjs.com/package/nodeshift)部署一个本地目录:

```
npx nodeshift --dockerImage=registry.access.redhat.com/rhel8-beta/nodejs-14 --expose

```

## Node.js 入门

我介绍了 Node.js 14 中现有的几个 JavaScript 语言特性。我还向您展示了在 OpenShift 上开始使用 Node.js 14 的两种简单方法。要了解关于使用 Node.js 的更多信息，请查看 Lucas Holmquist 的“OpenShift 上的现代 web 应用程序”系列文章:

*   OpenShift 上的现代网络应用:第一部分——两个命令中的网络应用
*   OpenShift 上的现代 web 应用:第 2 部分——使用链式构建
*   OpenShift 上的现代 web 应用:第 3 部分——作为开发环境的 open shift

更多关于 Node.js 14 的改进和特性，请参见 [Node.js 14 官方公告](https://medium.com/@nodejs/node-js-version-14-available-now-8170d384567e)。