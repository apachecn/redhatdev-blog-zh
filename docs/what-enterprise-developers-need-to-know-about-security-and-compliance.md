# 关于安全性和合规性，企业开发人员需要了解什么

> 原文：<https://developers.redhat.com/blog/2020/06/23/what-enterprise-developers-need-to-know-about-security-and-compliance>

我工作中的一大乐事是，我可以与美国联邦和州政府机构雇用的一系列 IT 人员交谈和共事。这个范围包括[开发人员](https://developers.redhat.com/topics/devops/)工程师、开发人员、系统管理员、数据库管理员和[安全人员](https://developers.redhat.com/topics/security/)。与我交谈过的每个人，甚至是安全专业人员，都说 IT 安全性和法规遵从性可能是不精确的、主观的、势不可挡的和可变的——尤其是在联邦政府中。

过多的政策、法律和标准加在一起可能会令人生畏。下面是一个简短的列表:

*   运营授权(ATO)
*   联邦信息安全管理法案
*   联邦风险和授权管理计划(FedRAMP)
*   国防部云计算安全需求指南(国防部 SRG)
*   508 合规性

此外，*安全*或*合规*的含义因机构而异，甚至在一个机构内的授权官员之间也有所不同，这取决于一系列因素。

许多开发人员询问更新的技术和行为，如基础设施即代码(IaC)、[持续集成/持续交付(CI/CD)](https://developers.redhat.com/topics/ci-cd/) 、[容器](https://developers.redhat.com/topics/containers/)以及一系列云服务如何映射到合规性框架和联邦安全法。作为一名开发人员，您经常想知道从软件开发生命周期的开始起您的安全责任是什么，并且您可能一直想知道到生产。相信我，你不是唯一一个感到疑惑的人。

本文致力于帮助开发人员理解更多的标准，从而减少未知和可变因素。最终目标是使安全性更加精确，建立已知的责任(和责任中的差距)，并将安全性融入到您的日常工作流中——即使需求和对它们的解释在不同的项目中有所变化。

## 尽可能地分担责任和继承遗产

首先，你的工作不是涵盖一切(或者，换句话说，不要试图煮沸海洋)。作为企业中的开发人员，您不可能交付安全代码、配置操作系统映像、监控网络和扫描文件系统。如果你尝试做所有这些，你将花费更少的时间来保护你所负责的组件——源代码、应用程序依赖和测试——并且你将会把自己分散得太分散。

为服务和产品提供可靠、有信誉的选择的平台和供应商的数量从未像现在这样多，而且还在增长。基础设施云提供商提供计算、网络和存储供应等资源，而软件供应商通常为其产品提供受支持的容器映像。

这里的教训是继承这些平台和供应商承担的保护信息系统的责任。这样做可以让您在 ATO 流程中检查大量的安全性控制。在补丁或响应安全事件的情况下，这些提供商(不仅仅是您)有责任提供补丁。

这并不意味着您应该停止与 ops 和 infosec 的对话，或者如果他们提供的内部企业服务失败了，您可以将所有责任都推到他们身上。如果有的话，这意味着您应该与欢迎您合作的所有类型的提供商合作。有意分担管理风险的责任。拥有一个已建立的 [RACI 矩阵](https://www.cio.com/article/2395825/project-management-how-to-design-a-successful-raci-project-plan.html)支持开发人员、运营和安全团队之间清晰的沟通和责任分配。它强制执行生成成功的 DevSecOps 文化所必需的行为。

## 拥抱持续安全的世界

如果您仍然怀疑什么时候关注您所负责区域的安全是重要的，那么就一直关注它。相反，要持续不断地做。

作者 Gene Kim、Nicole Forsgren 和 Jez Humble 通过他们的书*[Accelerate:Building and Scaling High performance Technology Organizations](https://itrevolution.com/book/accelerate/)*，为参与企业软件交付的每个人帮了大忙，这本书测量了导致整体组织绩效的行为。使用从所有类型的企业收集的数据，这本书证明了像持续交付和持续安全这样的行为会导致更协作的文化和整体提高的组织绩效。正如书中所说:

> “在工作中建立安全性的团队在持续交付方面也做得更好。”

换句话说，如果您将安全性构建到日常工作中，您的组织将实现其目标并交付更好的软件。持续执行安全活动和行为迫使您边走边学。它还迫使您放弃不良行为，比如将 SSH 密钥放在存储库中。

由美国国防部编写的 [DevSecOps 参考设计](https://dodcio.defense.gov/Portals/0/Documents/DoD%20Enterprise%20DevSecOps%20Reference%20Design%20v1.0_Public%20Release.pdf?ver=2019-09-26-115824-583)是一个极好的资源，用于确定在特定的软件开发生命周期阶段应用哪些安全步骤来支持持续的安全性。除其他外，该指南建议:

*   在开发人员编写源代码时扫描源代码的 IDE 安全插件。
*   提交前扫描代码的静态源代码工具。
*   源代码存储库安全插件，用于检查授权令牌、SSH 密钥和密码等内容。
*   带有安全扫描框架的容器注册中心可以识别容器图像中的漏洞。
*   对使用潜在易受攻击的开源库的应用程序进行依赖性检查。
*   动态和交互式工具，在应用程序构建后对其进行扫描。
*   模拟攻击者的手动渗透测试，可能会发现其他工具无法发现的漏洞。
*   持续监控解决方案，可在运行时识别威胁，并持续记录事件以供分析。

## 参与选择您的工具

告诉开发人员对工具要挑剔是不言而喻的。然而，我和太多认为信息安全是别人的工作的开发人员交谈过。这是每个人的工作。

因为开发人员对安全性负责，所以在开发、构建和测试阶段选择安全产品非常重要。在 *Accelerate* 中，作者 Kim、Forsgren 和 Humble 认为选择正确的工具是一项技术工作。“当所提供的工具确实使使用它们的工程师的生活变得更容易时，他们会出于自己的自由意志采用它们。”

如果安全性被您现有工作的精确性所包围，例如您最喜欢的 ide 的安全性插件或扫描工具，它们只是您 CI/CD 管道中的另一个组件，那么安全性将会更加精确。如果一个工具给你更快的反馈，如果它比在交付周期的后期修复一个漏洞花费更少的时间，那么你更倾向于使用它。选择合适的工具将安全性融入您的日常工作中。

## 技术在发展，但安全原则却没有

容器编排、[无服务器](https://developers.redhat.com/topics/serverless-architecture/)平台、智能安全工具和自动化部署可能会改变我们在生产中扫描漏洞或修补应用程序的方式，但这不会改变这样一个事实，即[安全仍然更多地是关于过程而不是产品](https://www.schneier.com/essays/archives/2000/04/the_process_of_secur.html)，或者[深度防御仍然是保护信息系统的至高无上的](https://www.csoonline.com/article/3268066/how-important-defense-in-depth-will-be-as-the-lines-between-security-layers-blur.html)。有一个很好的理由说明[美国国家标准与技术研究所关于安全控制的专门出版物](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf)包括 18 个不同的安全控制系列或类别。从如何加密敏感信息到将灭火器放在哪里，一切都关系到信息安全。密码术和火灾管理的新技术一直在出现。

如果有的话，最佳实践是对新技术持开放态度，这些新技术可以帮助您解决组织中的安全问题。自动化配置管理、[容器编排](https://developers.redhat.com/openshift/)和[不可变基础设施](https://www.infosecurity-magazine.com/opinions/mmutable-infrastructure-modern/)都使得一致且可重复地配置信息系统变得更加容易。IDE 安全插件和容器注册表安全扫描有助于将安全性“左移”,使开发人员能够及早发现漏洞，而不是在下游进行安全检查时发现。

这些工具并没有消除对安全确认管理和漏洞扫描等标准安全原则的需求；相反，它们让你把注意力集中在最需要分析和管理风险的地方。

## 教育是银弹

就像软件安全和交付一样，学习应该是持续的。新工具有助于管理安全预防和响应，但攻击者也在不断创新和学习。有大量的资源可以安全地交付软件，尤其是在[敏捷](https://www.amazon.com/Agile-Application-Security-Enabling-Continuous/dp/1491938846/ref=sr_1_1?qid=1582580009&refinements=p_27%3ALaura+Bell&s=books&sr=1-1&text=Laura+Bell)和[开发运维](https://www.amazon.com/DevOps-Handbook-World-Class-Reliability-Organizations/dp/1942788002/ref=pd_sbs_14_t_1/130-8957903-8014362?_encoding=UTF8&pd_rd_i=1942788002&pd_rd_r=9686a1c0-4eed-447d-873d-f6ee8b9a3f2b&pd_rd_w=N7EZv&pd_rd_wg=vdLqs&pf_rd_p=5cfcfe89-300f-47d2-b1ad-a4e27203a02a&pf_rd_r=GCBG5QNC33DEPJVXBCQ7&psc=1&refRID=GCBG5QNC33DEPJVXBCQ7)的背景下。作为一名前产品经理，我不能充分陈述[威胁建模](https://www.amazon.com/Threat-Modeling-Designing-Adam-Shostack-ebook/dp/B00IG71FAS)的力量，作为 scrum 团队在软件开发生命周期早期识别潜在风险的练习。

我还建议借鉴以前的成功经验并复制结果。阅读关于其他人如何持续提供安全的研究，比如国家地理空间情报局的 Red Hat OpenShift 的成功。在这个项目中，应用能够从一个强化的 CI/CD 管道和容器平台中继承大约 90%的安全控制，这不仅大大减少了开发人员在应用中处理控制的责任，还减少了合规时间(在一个敏捷 sprint 中的应用 ATOs！)更好的是，那些追求“持续授权”的信息安全组织，如[美国空军](https://www.fedscoop.com/fast-track-ato-air-force-wanda-jones-heath/)，可以真正补充甚至加速他们并行进行持续安全的团队(即所有的船都朝着同一个方向划)。

反过来，我也会看看过去的安全事件，例如 2017 年的 [Apache Struts 关于 Equifax 安全漏洞](https://blogs.apache.org/foundation/entry/apache-struts-statement-on-equifax) [的声明。使用这些过去的事件来确定用户和企业哪里出错了。仔细检查它们，了解不分担责任和采用持续安全性的后果。](https://youtu.be/OO4UVCp5QIg?t=876)

Apache 声明中的建议适用于所有类型的软件交付，不仅仅是使用 Struts 的项目，当然也不仅仅是 Equifax。今天每个人都在构建软件，包括政府部门和各种类型的机构。为了持续安全地交付软件，安全需要是每个人的工作。这种类型的行为导致更低的成本和更快的实现时间，这必然会导致更快地批准法规遵从性和安全授权审核。让我们把它作为软件开发生命周期每个阶段不可分割的一部分。

*Last updated: June 24, 2020*