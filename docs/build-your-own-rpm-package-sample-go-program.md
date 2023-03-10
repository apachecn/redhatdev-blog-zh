# 用一个样例 Go 程序构建自己的 RPM 包

> 原文：<https://developers.redhat.com/articles/2021/05/21/build-your-own-rpm-package-sample-go-program>

部署通常涉及多个棘手的步骤。如今，我们有各种各样的工具来帮助我们创建可重复的部署。在本文中，我将向您展示构建一个基本的 RPM 包是多么容易。

我们有包装经理已经有一段时间了。 [RPM](https://rpm.org/) 和 [YUM](http://yum.baseurl.org/) 简化了软件的安装、更新或移除。然而，许多公司只使用软件包管理器来安装来自操作系统供应商的软件，而不使用它们进行部署。一开始创建一个包可能会令人生畏，但通常，这是一个有益的练习，可以简化您的管道。作为测试用例，我将向您展示如何打包一个用 [Go](https://golang.org/) 编写的简单程序。

## 创建包

许多站点依赖配置管理器进行部署。例如，一个典型的[可回答的](/products/ansible/overview)剧本可能是:

```
 tasks:
    - name: 'Copy the artifact'
      copy:
        src: 'my_app'
        dest: '/usr/bin/my_app'

    - name: 'Copy configuration files'
      template:
        src: config.json
        dest: /etc/my_app/config.json
```

当然，现实生活中的剧本将包括更多的步骤，如检查以前的安装或处理服务。但是为什么不用这样的东西呢？

```
  tasks:
    - name: 'Install my_app'
      yum:
        name: 'my_app'
```

现在，让我们看看提供网页的 Go 应用程序。下面是`main.go`文件:

```
package main

import (
    "encoding/json"
    "flag"
    "fmt"
    "io/ioutil"
    "log"
    "net/http"
)

type config struct {
    Text string `json:"string"`
}

func main() {

    var filename = flag.String("config", "config.json", "")
    flag.Parse()

    data, err := ioutil.ReadFile(*filename)
    if err != nil {
        log.Fatalln(err)
    }

    var config config
    err = json.Unmarshal(data, &config)

    if err != nil {
        log.Fatalln(err)
    }

    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, config.Text)
    })

    log.Fatal(http.ListenAndServe(":8081", nil))

}
```

还有我们的`config.json`:

```
{
    "string": "Hello world :)"
}
```

如果我们运行这个程序，我们应该在端口 8081 看到一个带有`config.json`文本的网页。它离生产就绪还差得很远，但它将作为一个例子。

## 添加服务

服务呢？添加服务是统一应用程序管理的一个很好的方式，所以让我们创建我们的`my_app.service`:

```
[Unit]
Description=My App

[Service]
Type=simple
ExecStart=/usr/bin/my_app -config /etc/my_app/config.json

[Install]
WantedBy=multi-user.target
```

每当我们想要部署应用程序时，我们都需要:

1.  编译项目。
2.  复制到`/usr/bin/my_app`。
3.  将`config.json`文件复制到`/etc/my_app/config.json`。
4.  将`my_app.service`复制到`/etc/systemd/system/`。
5.  启动服务。

## 创建等级库文件

像 Ansible 一样，RPM 包需要一个定义文件，我们可以在其中指定安装步骤、依赖项以及在服务器上安装应用程序可能需要的其他内容:

```
$ sudo dnf install git
$ sudo dnf module install go-toolset
$ sudo dnf groupinstall "RPM Development Tools"
```

安装好所有这些之后，我们就可以创建包定义文件了，也称为*规范文件*:

```
$ rpmdev-newspec my_app.spec
```

规格文件可能很复杂，但我们将保持简单，以了解该工具的强大功能:

```
Name:           my_app
Version:        1.0
Release:        1%{?dist}
Summary:        A simple web app

License:        GPLv3
Source0:        %{name}-%{version}.tar.gz

BuildRequires:  golang
BuildRequires:  systemd-rpm-macros

Provides:       %{name} = %{version}

%description
A simple web app

%global debug_package %{nil}

%prep
%autosetup

%build
go build -v -o %{name}

%install
install -Dpm 0755 %{name} %{buildroot}%{_bindir}/%{name}
install -Dpm 0755 config.json %{buildroot}%{_sysconfdir}/%{name}/config.json
install -Dpm 644 %{name}.service %{buildroot}%{_unitdir}/%{name}.service

%check
# go test should be here... :)

%post
%systemd_post %{name}.service

%preun
%systemd_preun %{name}.service

%files
%dir %{_sysconfdir}/%{name}
%{_bindir}/%{name}
%{_unitdir}/%{name}.service
%config(noreplace) %{_sysconfdir}/%{name}/config.json

%changelog
* Wed May 19 2021 John Doe - 1.0-1
- First release%changelog 
```

一些注意事项:

*   `Source0`条目可以是源代码库，类似于:`https://github.com/*user*/my_app/archive/v%*version*.tar.gz`。
*   如果您在`Source0`中使用 URL，您可以发出`spectool -g my_app.spec`来下载您的源代码。
*   Git 允许您快速设置一个 tarball，而无需创建远程存储库:

    ```
    $ git archive --format=tar.gz --prefix=my_app-1.0/ -o my_app-1.0.tar.gz HEAD
    ```

*   tarball 内容应该是这样的:

    ```
     $tar tf my_app-1.0.tar.gz 
    my_app-1.0/
    my_app-1.0/config.json
    my_app-1.0/main.go
    my_app-1.0/my_app.service
    ```

## 构建 RPM

首先，我们需要创建`rpmbuild`结构，并将我们的 tarball 放在源文件的目录中:

```
$ rpmdev-setuptree
$ mv my_app-1.0.tar.gz ~/rpmbuild/SOURCES
```

现在，让我们为 [Red Hat Enterprise Linux](/products/rhel/overview) 8 构建 RPM:

```
$ rpmbuild -ba my_app.spec
```

就是这样！

您现在应该能够安装 RPM 并启动服务了:

```
$ sudo dnf install ~/rpmbuild/RPMS/x86_64/my_app-1.0-1.el8.x86_64.rpm
$ sudo systemctl start my_app
$ curl -L http://localhost:8081
```

你应该看到我们`config.json`的内容(顺便说一下，在`/etc/my_app`下面)。

但是如果我们有一个新版本的应用程序呢？我们只需要增加规范文件版本并重新构建它。DNF 将看到有新的更新可用。

如果你使用的是包仓库，你只需要运行`dnf update my_app`。

## 结论

如果你想更深入地了解将 RPM 文件合并到你的部署中的想法，我建议看看 [RPM 打包指南](https://rpm-packaging-guide.github.io)和[Fedora 打包指南](https://docs.fedoraproject.org/en-US/packaging-guidelines)。

此外，各种令人兴奋的工具可用于帮助构建过程，甚至创建供您使用的存储库，如 [mock](https://github.com/rpm-software-management/mock) 、 [fedpkg](https://pagure.io/fedpkg) 、 [COPR](https://pagure.io/copr/copr) 和 [Koji](https://pagure.io/koji/) 。这些工具可以在具有多个依赖项、复杂步骤或多个架构的复杂场景中为您提供帮助。

请注意，本文中演示的工作流也适用于 Fedora 和 CentOS Stream。

*Last updated: August 26, 2022*