<img  src= "https://gitee.com/wwinter117/pictures/raw/master/comments/pic_002.png"/>
更加详细介绍可以参考[官方文档](https://docs.docker.com/get-started/overview/)

# Docker概述

>Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

简单来说就是Docker就是一个平台（容器平台），方便开发、迁移以及运行我们的应用软件。

将软件与基础设施分离，独立于底层的基础设施，也就是说可以在Linux、Windows、VM等等上运行，解决因为使用的底层系统不同而导致的程序无法运行或运行异常，方便软件进行迁移。

并且迁移、测试、部署应用十分快速，提升工作效率，节约时间成本。

# Docker平台

>Docker provides the ability to package and run an application in a loosely isolated environment called a container. The isolation and security allows you to run many containers simultaneously on a given host. Containers are lightweight and contain everything needed to run the application, so you do not need to rely on what is currently installed on the host. You can easily share containers while you work, and be sure that everyone you share with gets the same container that works in the same way

Docker让我们可以在容器中打包、运行程序，而容器具有隔离性，因此可以安全的在一个主机上同时运行多个容器，并且每个容器包含运行程序所需要的环境，系统工具，系统库等，可以独立运行，不依赖于底层设施。



# Docker特点

- 快速一致的交付应用程序（标准）
- 响应式部署和扩展
- 同样的硬件下可以运行更多的程序，负载量大（轻量）

# Docker架构

<img src="https://gitee.com/wwinter117/pictures/raw/master/comments/Docker-Architecture.png"/>


**[回到目录](../../README.md)**
