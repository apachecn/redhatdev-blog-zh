# 红帽峰会 2018 Burr Sutter 演示-多云

> 原文：<https://developers.redhat.com/blog/2018/05/10/red-hat-summit-2018-burr-sutter-demo>

红帽峰会 2018 的一个亮点是 Burr Sutter ( [@burrsutter](https://twitter.com/burrsutter) )和一个开发团队的另一个现场演示。该演示特别引人入胜，因为观众在他们的手机上使用一款移动游戏进行参与，该游戏与 Burr 团队开发的云计算后端进行通信。演示的目的是展示这些技术，并展示如何用现代方法解决复杂的开发和部署挑战。

作为游戏的一部分，观众被要求对所要求的物体拍照。根据照片对请求的表现打分。这些照片被自动上传到云中，在 OpenShift 上运行的 TensorFlow 图像识别服务使用机器学习对每张照片进行评分。(视频在休息后提供。)

演示的主要想法是应用程序运行在三个不同的云环境中，Amazon Web Services (AWS)、Azure 和私有云。使用红帽存储，上传的照片在所有三个云平台上同步。在游戏进行到一半时，他们终止了运行在 AWS 上的服务。观众能够实时看到多云故障转移。证据是，作为该系统的用户，观众可以从他们的手机上看到他们在两个剩余的云中重新平衡，并且所有上传的数据仍然可用。

最主要的一点是，尽管在云中运行，但它是你的应用程序和数据的*，你应该能够在最适合你的任何地方运行它。正确的平台选择为您提供了灵活性，同时避免了锁定。部署到两个(或更多)不同的云平台以确保可用性可以轻松实现。这就是人们现在所说的多重云。*

 *当然，该演示基于许多最新技术，包括:

*   Eclipse Che、OpenShift.io 等云原生开发工具
*   使用 Vert.x 和 Red Hat OpenShift 应用运行时构建的反应式数据管道
*   用 Istio 管理的服务网格
*   无服务器计算:在 Red Hat OpenShift 上运行 Tensorflow 作为服务的机器学习。

你可以在新设计的[developers.redhat.com](https://developers.redhat.com/)上阅读这些技术，观看视频，并尝试开发工具。敬请关注红帽峰会 2018 新增内容。

[https://www.youtube.com/embed/hu2BmE1Wk_Q?start=396](https://www.youtube.com/embed/hu2BmE1Wk_Q?start=396)

你可以在 redhat.com/summit 观看所有的主题视频。上面的演示来自周四上午的主题演讲。伯尔的部分从视频的 6:36 开始。更多视频和采访可在 thecube.net/red-hat-summit-2018 的[上获得。](https://www.thecube.net/red-hat-summit-2018)

*Last updated: May 13, 2018**