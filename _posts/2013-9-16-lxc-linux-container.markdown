---
layout: post
title: LXC介绍
---

LXC(Linux containers): Linux容器. 网上介绍都说lxc是一种虚拟化技术. 但是我觉得lxc更多的是一种资源的控制和隔离工具.

> The linux containers, lxc, aims to use these new functionalities to pro-
> vide an userspace container object which provides full resource isolation
> and resource control for an applications or a system.

以上是[lxc官方](https://github.com/lxc/lxc)的介绍. 可见lxc是一种将系统资源按照类型和需求分割给多个对象独立使用, 对象之间保持隔离的工具. 系统资源通常指CPU、内存、网卡、磁盘等.

由于lxc并不需要对硬件进行模拟并且lxc容器与主机共享内核和一些其它资源,  这也使得lxc非常的轻量级. 


使用lxc运行应用有以下好处:

* **安全** 因为应用都运行在被隔离的各个不同容器中, 一个容器或容器中的应用出现问题, 并不会影响到其它容器.
* **便携性** 容器可以打包并部署到其它相同cpu架构的机器上.
* **可控性** 容器能使用的资源都可以进行限制.

当然lxc的机制也有不足的地方.  
lxc的使用场景比较适合应用的运行, 而不适合需要完全掌控系统的场景. 另外, lxc不支持不同的cpu体系结构. 也就是说64位的容器不能运行在32位机器上.

lxc对资源的控制和隔离是基于chroot和Cgroups的.

chroot(change root directory). 在 linux系统中, 默认的目录结构都是以根(root) 开始的. 而使用 chroot, 可以设置将系统的目录结构以指定的位置做为'/'的位置.  因为linux中一切都是基于文件的(比如/proc 就是所有的进程所在的目录). 所以可以说, chroot相当于启动了一个非常简单的被隔离的容器. 如果再为每个容器加上自己的上下文,进程和网络, 那么就可以做到真正的资源隔离了.

Cgroups(control groups)是Linux内核提供的一种可以限制,记录,隔离进程组(process groups)所使用的物理资源(如:cpu,内存,IO等等)的机制.
Cgroups提供了以下功能:

1. 限制进程组可以使用的资源数量
2. 进程组的优先级控制
3. 记录进程组使用的资源数量
4. 进程组隔离
5. 进程组控制
