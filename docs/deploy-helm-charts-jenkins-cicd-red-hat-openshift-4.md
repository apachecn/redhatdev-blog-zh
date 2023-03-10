# 在 Red Hat OpenShift 4 中使用 Jenkins CI/CD 部署舵图

> 原文：<https://developers.redhat.com/articles/2021/05/24/deploy-helm-charts-jenkins-cicd-red-hat-openshift-4>

Helm 是 [Kubernetes](/topics/kubernetes/) 的包装经理。Helm 使用了一种叫做*图表*的打包格式，其中包含了部署应用所需的所有 Kubernetes 资源，比如部署、服务、入口等。掌舵图对于在 Kubernetes 集群上安装应用程序和执行升级非常有用。

在本文中，我将向您展示如何使用 Jenkins [持续集成和持续交付(CI/CD)](/topics/ci-cd/) 和 [Red Hat OpenShift 4](/products/openshift/overview) 部署一个 Helm 图表。图 1 显示了该流程的高级视图。

[![Diagram showing how a Helm chart is deployed to Red Hat OpenShift with Jenkins CI/CD.](img/d328afb45de18dcee908dea0371e677c.png)](/sites/default/files/blog/2021/02/jenkins.jpg)Deploying Helm charts with Jenkins CI/CD.

Figure 1: Deploying Helm charts with Jenkins CI/CD.

## 先决条件

在开始本练习之前，您需要安装以下技术:

*   [红帽 OpenShift 集装箱平台 4.6](https://docs.openshift.com/container-platform/4.6/architecture/architecture-installation.html)
*   [Helm 3 命令行界面(CLI)](https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/)

## 部署詹金斯

首先，使用模板或 OpenShift 上的开发人员目录安装 Jenkins:

1.  假设您的开发环境中有 OpenShift Container Platform 4，在 OpenShift web 控制台中导航到**开发者视角**。
2.  从开发者目录中选择**添加**并搜索**詹金斯**，然后点击**实例化模板**，如图 2 所示。

[![Screenshot showing how to install Jenkins using the Developer Catalog on Red Hat OpenShift.](img/3006ad003b033abd6ba729f6276f77d2.png)](/sites/default/files/blog/2021/02/Screenshot-2021-02-21-at-6.27.02-PM.png)

Figure 2: Install Jenkins using the Developer Catalog on OpenShift.

## 使用 Helm 客户端创建一个 Jenkins 代理映像

这是在容器中运行`helm` CLI 命令所必需的。

为了使用 Helm 客户端创建代理，我们将使用 [ose-jenkins-agent-base](https://catalog.redhat.com/software/containers/openshift4/ose-jenkins-agent-base/5cdd8e2fbed8bd5717d66e77) 作为基本映像:

1.  创建 Dockerfile 文件:

    ```
    FROM registry.redhat.io/openshift4/ose-jenkins-agent-base

    COPY helm /usr/bin/helm  ---> untar helm-linux-amd64.tar.gz(Helm 3 CLI) to get helm client. 
    RUN chmod +x /usr/bin/helm
    ```

2.  构建图像:

    ```
    podman build -t quay.io/<User-ID>/jenkins-helm-agent:v0.1 .
    ```

3.  将图像推送到存储库:

    ```
    podman push quay.io/<USER-ID>/jenkins-helm-agent:v0.1
    ```

**注意**:您也可以将图像推送到 [OpenShift 内部图像注册表](https://docs.openshift.com/container-platform/4.6/registry/accessing-the-registry.html)。

## 在 Jenkins 中配置 pod 模板

接下来，为新的 Helm 代理配置 pod 模板:

1.  打开詹金斯控制台。点击**管理詹金斯** **— >** **管理节点和云** **— >** **配置云**。
2.  为 Helm agent 添加新的 pod 模板，并在**名称**和**标签**字段中输入`helm`，如图 3 所示。

[![Screenshot showing how to add new pod templates for the Helm agent; "helm" has been entered in the Name and Labels fields.](img/5849fb76426090cf579fe155b63e35ab.png)](/sites/default/files/blog/2021/02/Screenshot-2021-02-21-at-8.56.48-PM.png)

Figure 3: Configuring the Helm agent in the pod template.

## 建立詹金斯管道阶段

构建管道阶段:

1.  Add the repo:

    ```
    stage("add Repo") {
                        steps {
                               sh "helm repo add shailendra ${repo}"
                            }
                       }
    ```

2.  将图表部署到开发人员/UAT:

    ```
    stage("Deploy to Dev") {
                            steps {
                                script{
    					openshift.withCluster(){
                                            sh "helm upgrade --install my-guestbook shailendra/guestbook --values dev/values.yaml -n dev --wait"
                                        }
                                    }
                                }
                        }
    ```

样本 [Jenkinsfile](https://github.com/shailendra14k/sample-helm-chart/blob/master/Jenkinsfile) :

```
def repo="https://shailendra14k.github.io/sample-helm-chart/"
pipeline{
		agent{
			label 'helm'
		}
		stages{

                stage("add Repo") {
                        steps {
                               sh "helm repo add shailendra ${repo}"
                            }
                    }
				stage("Deploy to Dev") {
                        steps {
                            script{
					openshift.withCluster(){
                                        sh "helm upgrade --install helm-app shailendra/sample-app --values dev/values.yaml -n dev --wait"
                                    }
                                }
                            }
                    }
                stage("Deploy to UAT") {
                        steps {
                            script{
					openshift.withCluster(){
                                        sh "helm upgrade --install helm-app shailendra/sample-app --values uat/values.yaml -n uat --wait"
                                    }
                                }
                            }
                    }
            }
        }
```

## 部署管道

现在，您可以部署管道了:

1.  打开詹金斯控制台，点击**新项目**。
2.  输入名称并选择**管道**。
3.  转到**管道**选项卡。
4.  从 SCM 中选择**管道脚本，并提供指向 Jenkinsfile: `https://github.com/shailendra14k/sample-helm-chart.git`的 GitHub 链接。参见图 4。**

[![Screenshot showing the Pipeline tab. Select Pipeline script from SCM is selected and the GitHub link that points to the Jenkinsfile is specified.](img/e44b48474760d04bfb46ba4e2f71bdd3.png)](/sites/default/files/blog/2021/02/Screenshot-2021-02-22-at-12.34.42-AM.png)

Figure 4: Adding the GitHub link to the Jenkinsfile in the Pipeline tab.

## 运行并验证管道

最后，运行并验证管道:

1.  点击 **Build Now** 运行管道(见图 5)，然后验证输出。[![Screenshot of the Jenkins console described in this section.](img/7e788f4574aa6b5fb7aa8651c48ae960.png)](/sites/default/files/blog/2021/02/Screenshot-2021-02-22-at-1.11.22-AM.png)

    图 5:运行管道。

2.  验证`DEV`名称空间上的应用程序部署:

    ```
    oc get all -n dev
    NAME                                       READY   STATUS    RESTARTS   AGE
    pod/helm-app-sample-app-7b6bd8f757-bqzq7   1/1     Running   0          92s

    NAME                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    service/helm-app-sample-app   ClusterIP   172.30.20.99           6379/TCP   92s

    NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/helm-app-sample-app   1/1     1            1           92s

    NAME                                             DESIRED   CURRENT   READY   AGE
    replicaset.apps/helm-app-sample-app-7b6bd8f757   1         1         1       93s
    ```

3.  查看舵图状态:

    ```
    helm list -A
    NAME    	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART           	APP VERSION
    helm-app    	dev      	2       	2021-02-21 19:40:41.251103914 +0000 UTC	deployed	sample-app-0.1.0	1.16.0
    helm-app    	uat      	2       	2021-02-21 19:40:47.85977876 +0000 UTC 	deployed	sample-app-0.1.0	1.16.0
    ```

## 结论

感谢阅读！希望这篇文章能帮助你入门 OpenShift 上的 Helm CI/CD。

*Last updated: August 24, 2022*