---
title: Docker中遗失的包
date: 2014-04-22
---
[原文](http://crosbymichael.com/the-lost-packages-of-docker.html)   [翻译](http://www.oschina.net/translate/the-lost-packages-of-docker)

## netlink

我们有一个Netlink的纯Go实现，你可以用在你的项目中。你可以使用这个包来创建veth 接口，桥，设置IP，mtu，和网络接口的其他设置，移动网络接口到不同的Linux命名空间中，等等一堆事情。创建veth pairs和对你的每个docker容器分配一个IP的代码是相同的。

让我们用这个包创建一个桥并设置一个IP：

```
package main

import (
        "github.com/dotcloud/docker/pkg/netlink"
        "log"
        "net"
)

func main() {
        // create a new bridge
        if err := netlink.CreateBridge("mydocker0", false); err != nil {
                log.Fatal(err)
        }
        // get the bridge
        bridge, err := net.InterfaceByName("mydocker0")
        if err != nil {
                log.Fatal(err)
        }

        ip, ipNet, err := net.ParseCIDR("10.0.41.1/16")
        if err != nil {
                log.Fatal(err)
        }

        // add an ip to the bridge
        if err := netlink.NetworkLinkAddIp(bridge, ip, ipNet); err != nil {
                log.Fatal(err)
        }
        // bring the interface up
        if err := netlink.NetworkLinkUp(bridge); err != nil {
                log.Fatal(err)
        }
}
```

我们可以运行上面的应用，然后运行 ip a 来查看我们的新接口：

```
2: mydocker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether ae:1f:cb:af:f2:54 brd ff:ff:ff:ff:ff:ff
    inet 10.0.41.1/16 scope global mydocker0
    valid_lft forever preferred_lft forever
    inet6 fe80::ac1f:cbff:feaf:f254/64 scope link
    valid_lft forever preferred_lft forever
```

netlink 包功能还不齐全，但希望正在于未知，我们可以一起工作使其变得更好。

## user

Linux的用户和组函数不依赖于外部库。

## graphdb

graphdb是一个构建在SQLite之上的极小图数据库，通过节点命名以及它们是怎样关联的。它实现了大多数图数据库拥有的一个小子集，但是提供一个简单的接口去表示节点间的关系。这用于容器的命名和docker之间的连接。

```
package main

import (
        "github.com/dotcloud/docker/pkg/graphdb"
        "log"
)

func main() {
        db, err := graphdb.NewSqliteConn("/root/links.db")
        if err != nil {
                log.Fatal(err)
        }
        defer db.Close()

        if _, err := db.Set("/parent", "momma"); err != nil {
                log.Fatal(err)
        }

        child, err := db.Set("/child", "matt")
        if err != nil {
                log.Fatal(err)
        }
        otherKid, err := db.Set("/otherKid", "koye")
        if err != nil {
            log.Fatal(err)
        }

        // set entity with id of matt to child of the parent
        if _, err := db.Set("/parent/child", child.ID()); err != nil {
                log.Fatal(err)
        }
        if _, err := db.Set("/parent/kidbyaothername", otherKid.ID()); err != nil {
                log.Fatal(err)
        }

        // get all children for the key /parent
        for _, e := range db.List("/parent", -1) {
                log.Println(e.ID())
        }
}

:::bash
~|⇒ go run graph.go
2014/04/14 16:16:19 matt
2014/04/14 16:16:19 koye
```

## listenbuffer

Listenbuffer允许你立即去开启监听tcp、udp套接字，但是 在接受连接之前，需要等待你的应用程序发出信号。
在docker中使用Listenbuffer，所以你开启一个守护进程和一个客户请求，在守护进程完成加载之前，一个初始化的脚本被发送。这确保了初始化脚本不能接收一个错误，而你的服务器却能开始接收连接，当它准备好时。如果你有很长的引导，那么它使用内核积压去缓冲连接，这是一个提供一些值的超级简单封装。

## mount

挂载是一个帮助你在挂载了许多东西的前提下工作的包。它允许它的选项使用fstab风格，同时也提供挂载表分析函数，以便于你的应用程序很容易找到挂载点。

```
// do a readonly bind mount
if err := mount.Mount("/myfile", "/myotherfile", "none", "bind,ro"); err != nil {
    log.Fatal(err)
}
defer mount.Unmount("/myotherfile")
```

## term

Term处理终端大小，原始模式和其他的设置，比如与 其他linux,darwin或者BSD系统的Unix终端进行交互的。

## cgroups

现在这个包处于沉重发展时期，但是 通过原始的cgroup文件系统API或者 支持的系统API,它 为liccontainer提供了cgroup功能。

## libcontainer

libcontainer位于pkg文件中。它是运行docker容器的默认执行驱动：设定命名空间、网络、挂载、管理容器进程。你需要提供一个根文件系统和一个关于libcontainer怎样被支持去运行在容器中配置文件，然后完成剩余的工作。它允许产生新的容器或者附加到现有的容器上。

---

最后我想说为什么写这篇文章？我想要让人们知道这些包。然而其他项目能够区分git仓库，对于它们使用和开发的不同依赖， docker不得不把这些包放在它的主库里，从而导致许多人们不知道它们的存在。所以如果它们中的一些包引起你的兴趣，你可自由的在你的项目中使用、贡献、写文档、测试，让它们越来越好。
如果当前你知道在docker代码库中你所感兴趣的地方是一个不可重用的包，请提交一个“pull request”，使它成为可重用的。我们一直寻求使docker内核更小，我们的工作给他人带来便利。