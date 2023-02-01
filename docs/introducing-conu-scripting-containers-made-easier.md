# 引入 conu 脚本容器变得更加容易

> 原文：<https://developers.redhat.com/blog/2018/03/07/introducing-conu-scripting-containers-made-easier>

需要一个简单易用的处理程序来编写测试和其他围绕容器的代码，实现有用的方法和实用程序。为此，我们引入了一个底层的 [Python](https://www.python.org/) 库 [conu](http://conu.readthedocs.io/en/latest/) 。

这个项目从一开始就被容器维护者和测试者的需求所驱动。除了基本的映像和容器管理方法之外，它还提供了其他常用的功能，例如容器挂载、获取 IP 地址的快捷方式、公开的端口、日志、名称、使用[源到映像、](https://github.com/openshift/source-to-image)扩展映像等等。

conu 的目标是稳定的引擎无关的 API，这些 API 将由几个容器运行时后端实现。在两个不同的容器引擎之间切换应该只需要最小的努力。当用于测试时，可以为多个后端执行一组测试。

## 你好世界

在下面的例子中有一段代码，我们从指定的图像中运行一个容器，检查它的输出，然后优雅地删除。

我们已经决定我们想要的容器运行时应该是 docker(现在唯一完全实现的容器运行时)。使用 DockerRunBuilder 实例运行映像，这是为 docker 容器运行命令设置附加选项和自定义命令的方式。

```
import conu, logging

def check_output(image, message):
    command_build = conu.DockerRunBuilder(command=['echo', message])
    container = image.run_via_binary(command_build)

    try:
        # check_output
        assert container.logs_unicode() == message + '\n'
    finally:
        #cleanup
        container.stop()
        container.delete()

if __name__ == '__main__':
    with conu.DockerBackend(logging_level=logging.DEBUG) as backend:
        image = backend.ImageClass('registry.access.redhat.com/rhscl/httpd-24-rhel7')
        check_output(image, message='Hello World!')
```

## 获取 http 响应

当处理作为服务运行的容器时，容器状态“正在运行”通常是不够的。我们需要检查它的端口是否打开并准备好服务，还要向它发送定制请求。

```
def check_container_port(image):
    """
    run container and wait for successful
    response from the service exposed via port 8080
    """
    port=8080
    container = image.run_via_binary()
    container.wait_for_port(port)

    # check httpd runs
    http_response = container.http_request(port=port)
    assert http_response.ok

    # cleanup
    container.delete(force=True)
```

## 查看容器文件系统内部

为了检查配置文件的存在和内容，conu 提供了一种方法，通过一组预定义的有用方法轻松地挂载容器文件系统。装载是以只读模式进行的，但是我们计划在下一个版本中也实现读写模式。

```
def mount_container_filesystem(image):
    # run httpd container
    container = image.run_via_binary()

    # mount container filesystem
    with container.mount() as fs:
        # check presence of httpd configuration file
        assert fs.file_is_present('/etc/httpd/conf/httpd.conf')

        # check presence of default httpd index page
        index_path = ‘/opt/rh/httpd24/root/usr/share/httpd/noindex/index.html’
        assert fs.file_is_present(index_path)

        # and its content
        index_text = fs.read_file(index_path)
```

## 那么为什么不用 docker-py 呢？

除了 docker，conu 还旨在通过提供通用 API 来支持其他容器运行时。为了实现 docker 后端，conu 实际上使用了 docker-py。Conu 还实现了处理容器时通常使用的其他实用程序。采用其他实用程序应该也很简单。

## 那么容器测试框架呢？

你不必受限于指定的一组测试。当使用 conu 编写代码时，您可以获得端口、套接字和文件系统，并且您所拥有的唯一限制是 Python 设置的限制。如果 conu 不支持某些特性，并且您不想处理子进程，那么有一个 run_cmd 实用程序可以帮助您简单地运行所需的命令。

我们正在联系您收集反馈，并鼓励您为 conu 做出贡献，以使围绕容器的脚本编写更加高效。我们已经成功地使用 conu 进行了几次图像测试(例如这里的[和](https://github.com/container-images/postgresql/tree/master/test))，它还帮助实现了执行特定类型容器的客户端。

更多信息，参见[美国文件](http://conu.readthedocs.io/en/latest/)或[来源](https://github.com/fedora-modularity/conu)