# 在 CDK/Minishift 中用 xip.io 替换 nip.io 的步骤

> 原文：<https://developers.redhat.com/blog/2017/12/01/steps-replace-nip-io-xip-io-cdkminishift>

如果您是 Red Hat Container Development Kit(CDK)或 upstream Minishift 用户，您可能会受到 nip.io 不可用的影响。当您为 OpenShift(由 Minishift 提供)中运行的应用程序创建路由时，它会使用 nip.io 路由到 Minishift VM IP 地址。因此，无法访问使用 nip.io 后缀创建的路线。

可惜已经超过 24 小时了，nip.io 还没起来。所以下面是在 Minishift 中使用 xip.io 代替 nip.io 的步骤。

1.  获取 Minishift 虚拟机的 IP 地址。
    *   $ minishift ip
2.  要将路由后缀设置为 **xip.io** ，请在 ip-ADDRESS 后面运行以下命令，使用您在前面的命令中找到的实际 IP 地址。
    *   $ minishift openshift 配置集-patch“{ routing config”:{ " subdomain ":"<ip-address>. xip . io " } } '</ip-address>

详情请参考 Minishift 文档:[https://docs . open shift . org/latest/Minishift/open shift/open shift-client-binary . html # example-change-open shift-routing-suffix](https://docs.openshift.org/latest/minishift/openshift/openshift-client-binary.html#example-change-openshift-routing-suffix)