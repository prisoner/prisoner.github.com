---
date: 2013-10-22
title: CoreOS介绍
---

"Linux for Massive Server Deployments" 这是[CoreOS](http://coreos.com/)官方对自己的介绍. 
CoreOS被设计用做集群系统的大规模部署. 在linux世界里有大量的发行版可以做为服务器系统使用, 但是这些系统大多部署复杂, 更新系统更是困难重重. 这些都是CoreOS试图解决的问题.

CoreOS由几个部分组成:

* 最小化的Linux内核

CoreOS一个独特的设计是它拥有两个root分区. 一个被用作启动分区, 一个被用作更新分区. 系统更新将会被安装到更新分区, 并会自动切换. 在更新安装过程中, 为保证应用不会受到影响, CoreOS会使用Linux cgroups来限制磁盘 网络等IO使用. 详细请参考[http://coreos.com/using-coreos/updates/](http://coreos.com/using-coreos/updates/)

* 服务发现组件 [etcd](https://github.com/coreos/etcd)

etcd负责节点间的服务发现和配置共享. etcd的目的是在你构建的服务的地方增加更多的机器和服务自动扩展变得非常的容易.

* 容器管理 [docker](http://www.docker.io/)

CoreOS使用docker做为容器管理工具. CoreOS没有软件的包管理工具, 在CoreOS中运行的所有应用程序都要使用docker打包. 

* 系统服务管理 [systemd](http://zh.wikipedia.org/wiki/Systemd)

CoreOS使用systemd做为系统服务管理工具主要是因为下面几个原因:

1. 性能. 
2. 日志. systemd有现代化的日志功能.
3. 同时采用socket 式与D-Bus 总线式激活服务.

----

目前CoreOS并不是非常成熟, 如果想要尝试的话, 可以使用虚拟机进行安装实验.

1. 安装VirtualBox
2. 安装[Vagrant](/2013/08/30/vagrant.html)
3. 运行:

```sh
git clone https://github.com/coreos/coreos-vagrant/
cd coreos-vagrant
vagrant up
vagrant ssh
```

登陆就可以使用CoreOS了. 详细使用可以参考[官方手册](http://coreos.com/docs/vagrant/)

---

参考链接:

[服务器操作系统CoreOS初体验](http://www.blogjava.net/yongboy/archive/2013/08/26/403325.html)  
[CoreOS 中文手册](https://github.com/cloudcube/coreos-manual-chinese)  
[FIRST GLIMPSE AT COREOS](http://www.sebastien-han.fr/blog/2013/09/03/first-glimpse-at-coreos/)   
[Alex Polvi Explains CoreOS](http://www.activestate.com/blog/2013/08/alex-polvi-explains-coreos)