# 用 KubeAudit 对 Red Hat OpenShift 进行静态分析

> 原文：<https://developers.redhat.com/blog/2020/10/09/static-analysis-with-kubeaudit-for-red-hat-openshift>

在这篇文章中，我们介绍了一个新的实用程序，供那些希望确保他们的代码从上游 [Kubernetes](https://developers.redhat.com/topics/kubernetes) 干净地过渡到 [Red Hat OpenShift](https://developers.redhat.com/products/openshift/overview) 的开发人员使用。[OpenShift kube audit](https://github.com/AICoE/OpenShiftKubeAudit)(kube audit)是一个静态分析器，它从语义上检查用户代码的已知不兼容性，以便您可以在将代码引入 OpenShift 之前修复它们。KubeAudit 使用起来也很简单，很容易扩展。

## 运行审计

这是第一次发布，KubeAudit 目前只提供少量的审计，但是它们很容易编写。我们正在寻求来自社区的反馈和其他用例，以帮助使该工具更加全面。

首先，在 GitHub 上找到 [KubeAudit](https://github.com/AICoE/OpenShiftKubeAudit) 。

您需要在要审计的清单所在的同一台机器上的某个地方克隆 git repo，并设置一个 Python 虚拟环境:

```
$ git clone https://github.com/AICoE/OpenShiftKubeAudit.git
$ cd OpenShiftKubeAudit
$ python3 -m virtualenv venv
$ source venv/bin/activate
$ python3 -m pip install -r requirements.txt
```

现在，让我们运行审计工具:

```
$ ./audit.py /path/to/yaml/directory
```

它应该产生类似于以下内容的输出:

```
$ ./audit.py ./testaudits   
Audit Results:

Issue: Pod Security Policies in manifests
Severity: 1 - High
Resolution: In OpenShift PodSecurityPolicies are replaced by SecurityContextConstraints. See https://docs.openshift.com/container-platform/4.5/authentication/managing-security-context-constraints.html
Affected Files:
  Matched regex:
  testaudits/yaml/PodSecurityPolicies.yaml

Issue: NetworkPolicy has an IPBlock
Severity: 2 - Medium
Resolution: The Kubernetes v1 NetworkPolicy features are available in OpenShift Container Platform except for egress policy types and IPBlock.
Affected Files:
  Matched regex:
  testaudits/yaml/NetworkPolicies.yaml

Issue: Egress network policy set
Severity: 2 - Medium
Resolution: The Kubernetes v1 NetworkPolicy features are available in OpenShift Container Platform except for egress policy types and IPBlock.
Affected Files:
  Matched regex:
  testaudits/yaml/NetworkPolicies.yaml
```

现在确定了这些问题，开发人员可以在它们成为生产中的问题之前修复它们。然而，由于该工具是新的，有许多用例尚未确定，这就是该工具易于扩展的原因。

## 添加自定义审核

KubeAudit 使用简单的初始化(INI)语法来定义带有几个字段的审计:

*   **名称**:识别审计的名称或标题。
*   **查询(可选)**:这是一个 [jq 语法查询](https://stedolan.github.io/jq/)，审计工具使用它对清单的各个部分进行语义分析。注意，如果使用`regex`，则`query`是可选的。当您需要更全面的分析时，这种类型的查询非常有用。
*   **Regex(可选)**:您可以在每个文件中指定一个`regex`进行搜索。虽然`regex`比`query`更简单，但是实用程序可以忽略代码注释中的`regex` es。
*   **Regexes 部分(可选)**:当你需要多个`regex` es 时使用这个部分。该实用程序再次忽略在注释中找到的`regex`匹配，并按照结果出现的顺序匹配它们。如果所有的`regex`都以相应的顺序出现，就认为是匹配的。
*   **严重性**:该消息表示问题的相对严重性。它需要以数字开头，然后是严重性的文本描述。
*   **信息**:输出给用户的诊断信息。

下面是一个审计示例:

```
[Default]
# The name/title of the issue, e.g., "Pod Security Policies in manifests"
name = RunAsUser is set outside range, has some known issues
# A jq-syntax query to be applied to the yaml files. The query must return
# either True or False, but can be otherwise any valid jq string
query = ((.. | .runAsUser? | numbers) != 0) and ((.. | .runAsUser? | numbers) < 10000000 or (.. | .runAsUser? | numbers) > 20000000)
# Python-compatible regex to search for. The script automatically ignores
# commented lines
regex = runAsUser:
# Severity to help the user prioritize fixes
severity = 4 - Warning
# Message to output to the user, usually a resolution or more information
message = Setting runAsUser ID explicitly is not recommended. Additionally setting runAsUser explicitly outside of the expected range in OpenShift (10000000 - 20000000) has known incompatibilities

# This section can be used to construct a multiline regex for the files.
# Each regex is searched for in the order that it appears and automatically
# ignores comments. Any number of regexes can be added this way
[regexes]
regex1 =
regex2 =

```

## 结论

这个审计工具目前只支持 Kubernetes 清单，但我们计划将其扩展到包括[掌舵图](https://developers.redhat.com/blog/2020/07/20/advanced-helm-support-in-the-openshift-4-5-web-console/)和 [Go 代码](https://developers.redhat.com/blog/category/go/)。该工具还处于非常早期的阶段，但是正在寻找社区输入来帮助添加用例。

*Last updated: October 6, 2020*