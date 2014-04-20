---
date: 2013-10-16
title: 开源的Linux Container
---

最近比较关注Linux Container技术, 了解了几个开源的Linux Container. 下面做一些简单的介绍.

### Docker

![Docker](http://www.docker.io/static/img/docker-top-logo.png)

主页: [http://www.docker.io/](http://www.docker.io/)

Docker是一个可以将任何应用包装在LXC中运行的开源项目.  当应用被打包成Docker Image后, 部署和运维就变得极其简单. 可以使用统一的方式 来下载,启动,扩展,删除,迁移.

Docker的常用场景:

* 自动打包和部署应用
* 创建轻量级的私有PASS环境
* 自动测试和持续集成环境
* 部署和扩展web apps, databases 和 backend services

Docker使用Golang开发, 可以在github上找到它的[源代码](https://github.com/dotcloud/docker/).

### Dokku

主页: https://github.com/progrium/dokku

Dokku 是一个微型的 Heroku, 由 Docker 使用不多于 100 行的 Bash 编写.  完成安装后, 你就可以通过Git推送兼容Heroku 的应用到平台上运行. 该系统将使用 Heroku buildpacks 构建并在一个独立容器里运行, 最终结果就相当于是一个单机版的 Heroku.

### Warden

主页: https://github.com/cloudfoundry/warden

warden是cloudfoundry的核心组件之一.   warden的主要目的是提供一种简单的API来管理被隔离的容器，这种容器可以限制CPU, 内存, 存储, 网络等等资源. warden目前只支持linux. [这里](http://blog.csdn.net/k_james/article/details/8523934)有一篇使用介绍.

### lmctfy

主页: https://github.com/google/lmctfy

lmctfy是Google刚刚发布的Linux容器系统, 读音为lem-kut-fee. 目前项目还在密集开发中, 只提供了CPU与内存隔离. 按计划，lmctfy将提供磁盘IO和网络隔离, 名字空间, 根文件系统, 磁盘映像, 冻结层次和检查点恢复等, 规划还包括支持不同级别的服务质量, 监控和统计功能等.

### Cells

主页: http://systems.cs.columbia.edu/projects/cells/

Cells是一个用于在一个物理设备上运行多个智能手机或平板的基于LXC的项目.

