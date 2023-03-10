# 在 OpenShift 上使用 MySQL 部署 Spring Boot 应用程序

> 原文：<https://developers.redhat.com/blog/2018/03/27/spring-boot-mysql-openshift>

这篇文章展示了如何利用一个现有的使用 MySQL 的 [Spring Boot](http://projects.spring.io/spring-boot/) 独立项目，并将其部署在[红帽 OpenShift](https://developers.redhat.com/products/openshift/overview/) 上，在这个过程中，我们将创建 docker 映像，可以部署到大多数容器/云平台上。我将讨论创建 Dockerfile 文件，将容器映像推送到 OpenShift 注册表，最后创建部署了 Spring Boot 应用程序的 running pods。

为了在我的本地机器上使用 OpenShift 进行开发和测试，我使用了[*Red Hat Container Development Kit(CDK)*](https://developers.redhat.com/products/cdk/overview/)，它提供了一个在 Red Hat Enterprise Linux VM 上运行的基于 minishift 的单节点 OpenShift 集群。你可以在 Windows、macOS 或 Red Hat Enterprise Linux 上运行 CDK。为了测试，我使用了 Red Hat Enterprise Linux Workstation 7.3 版。它应该也能在 macOS 上运行。

为了创建 Spring Boot 应用程序，我使用了这篇文章作为指南。我使用现有的[openshift/mysql-56-centos 7](https://docs.openshift.com/container-platform/3.3/using_images/db_images/mysql.html)docker 映像将 MySQL 部署到 open shift。

你可以从我的个人 *[github repo 下载本文使用的代码。](https://github.com/1984shekhar/POC/tree/master/mysql-springboot-docker-openshift)* 在本文中，我将在本地构建容器映像，因此您需要能够使用 Maven 在本地构建项目。这个例子公开了一个 rest 服务，使用了 com.sample.app.MainController.java 的*。*

 *在存储库中，您会在`src/main/docker-files/`中找到一个 Dockerfile 文件。*[docker file _ spring boot _ MySQL](https://github.com/1984shekhar/POC/blob/master/mysql-springboot-docker-openshift/src/main/docker-files/dockerfile_springboot_mysql)*文件创建一个 docker 映像，以基于 *java8* *docker* 映像的 springboot 应用程序为基础。虽然这对于测试是可以的，但是对于生产部署，您可能希望使用基于 Red Hat Enterprise Linux 的映像。

构建应用程序:

1.使用`mvn clean install`构建项目。

2.将目标文件夹中生成的 jar 复制到`src/main/docker-files`。当创建 docker 映像时，可以在相同的位置找到应用程序 jar。

3.在`src/main/resources/application.properties`中设置数据库用户名、密码和 URL。注意:对于 OpenShift，建议将这些参数作为环境变量传递到容器中。

现在启动 CDK 虚拟机，让本地 OpenShift 集群运行起来。

1.使用 *minishift start* 启动 CDK 虚拟机:

```
$ minishift start
```

2.为 docker 和 oc CLI 设置本地环境:

```
$ eval $(minishift oc-env) 
$ eval $(minishift docker-env)
```

注意:上述 eval 命令在 Windows 上不起作用。有关更多信息，请参见 CDK 文档。

3.使用*开发者*账户登录 OpenShift 和 Docker:

```
$ oc login
$ docker login -u developer -p $(oc whoami -t) $(minishift openshift registry)
```

现在我们将构建容器图像。

1.将目录位置更改为项目中的`src/main/docker-files`。然后，执行以下命令来构建*容器*图像。注意:句号(。)位于 docker build 命令的末尾，用于指示当前目录:

```
$ docker build -t springboot_mysql -f ./dockerfile_springboot_mysql .
```

使用以下命令查看创建的容器映像:

```
$ docker images
```

2.将标签 springboot_mysql 添加到图像中，并将其推送到 OpenShift 注册表中:

```
$ docker tag springboot_mysql $(minishift openshift registry)/myproject/springboot_mysql
$ docker push $(minishift openshift registry)/myproject/springboot_mysql
```

3.接下来，提取 OpenShift *MySQL* 映像，并将其创建为 OpenShift 应用程序，该应用程序将初始化并运行它。更多信息请参考[文档](https://docs.openshift.com/container-platform/3.3/using_images/db_images/mysql.html):

```
docker pull openshift/mysql-56-centos7
oc new-app -e MYSQL_USER=root -e MYSQL_PASSWORD=root -e MYSQL_DATABASE=test openshift/mysql-56-centos7
```

4.等待运行 MySQL 的 pod 准备就绪。您可以使用`oc get pods`检查状态:

```
$ oc get pods
NAME READY STATUS RESTARTS AGE 
mysql-56-centos7-1-nvth9 1/1 Running 0 3m
```

5.接下来，ssh 到 *mysql* pod 并创建一个拥有完全权限的 MySQL root 用户:

```
$ oc rsh mysql-56-centos7-1-nvth9
sh-4.2$ mysql -u root
CREATE USER 'root'@'%' IDENTIFIED BY 'root';
Query OK, 0 rows affected (0.00 sec)

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

exit
```

6.最后，使用*imagestream*spring boot _ MySQL 初始化 Spring Boot 应用程序:

```
$ oc get svc
NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
mysql-56-centos7 172.30.145.88 none 3306/TCP 8m

$ oc new-app -e spring_datasource_url=jdbc:mysql://172.30.145.88:3306/test springboot_mysql
$ oc get pods
NAME READY STATUS RESTARTS AGE
mysql-56-centos7-1-nvth9 1/1 Running 0 12m
springbootmysql-1-5ngv4 1/1 Running 0 9m
```

7.检查 pod 日志:

```
oc logs -f springbootmysql-1-5ngv4
```

8.接下来，将服务公开为路由:

```
$ oc get svc
NAME CLUSTER-IP EXTERNAL-IP PORT(S) AGE
mysql-56-centos7 172.30.242.225 none 3306/TCP 14m
springbootmysql 172.30.207.116 none 8080/TCP 1m

$ oc expose svc springbootmysql
route "springbootmysql" exposed

$ oc get route
NAME HOST/PORT PATH SERVICES PORT TERMINATION WILDCARD
springbootmysql springbootmysql-myproject.192.168.42.182.nip.io springbootmysql 8080-tcp None
```

9.使用`curl`测试应用程序。您应该会看到数据库表中所有条目的列表:

```
$ curl -v http://springbootmysql-myproject.192.168.42.182.nip.io/demo/all
```

10.接下来，使用`curl`在数据库中创建一个条目:

```
$ curl http://springbootmysql-myproject.192.168.42.182.nip.io/demo/add?name=SpringBootMysqlTest
Saved
```

11.查看数据库中条目的更新列表:

```
$ curl http://springbootmysql-myproject.192.168.42.182.nip.io/demo/all

[{"name":"UBUNTU 17.10 LTS","lastaudit":1502409600000,"id":1},{"name":"RHEL 7","lastaudit":1500595200000,"id":2},{"name":"Solaris 11","lastaudit":1502582400000,"id":3},{"name":"SpringBootTest","lastaudit":1519603200000,"id":4},{"name":"SpringBootMysqlTest","lastaudit":1519603200000,"id":5}
```

就是这样！

我希望这篇文章对您将现有的 *spring-boot* 应用程序迁移到 OpenShift 有所帮助。请注意，在生产环境中，应该使用 Red Hat 支持的映像。本文档仅用于开发目的。它应该帮助你创建在容器中运行的 *spring-boot* 应用程序；以及如何在 OpenShift 中为 *spring-boot* 设置 *MySQL* 连接。

*Last updated: January 29, 2019**