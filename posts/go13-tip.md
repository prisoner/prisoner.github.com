---
title: "Go 1.3 Release Notes"
date: '2014-06-24'
description:
categories:

tags:

---

6月18日，在Go 1.2发布六个月之后，Go官方宣布正式发布Go 1.3。Go 1.3并没有引入新的语言功能，只是进行了功能改进，并修复了一些bug以及用户的反馈。新的版本开始支持DragonFly BSD、Solaris、Plan 9和Google的NaCl（ Native Client），且显著改进了链接器和编译器。低版本的Go语言用户无需更改任何代码即可升级到Go 1.3。



主要改进包括：


* 改进了内存模型。在缓冲的channel发送和接收数据时增加了一条规则，使缓冲的channel可以被用作一个简单的信号灯（Semaphore）。
* 不再支持Windows 2000。微软在2010年时就不再支持Windows 2000，现在Go 1.3也放弃对它的支持。
* 支持BSD和Solaris系统。Go 1.3开始支持DragonFly BSD、FreeBSD、NetBSD、OpenBSD、Plan 9、Solaris，但对这些系统的支持都有一些其他特殊要求，比如对FreeBSD的支持必须要求内核编译时配置COMPAT_FREEBSD32参数。
* 支持 Native Client 虚拟机架构。Go 1.3既可以在32位Inter架构处理器上( GOARCH=386 )运行，也能在64位Intel架构上运行，但是在64位架构上使用的是32位pointer，对于ARM架构暂不支持。关于Native Client的介绍可以阅读其官方介绍。
* 改进了栈的实现方式。将栈实现方式从分段（segmented）模型改为连续（contiguous）模型。当一个goroutine（ Go 语言提供的一种用户态线程）需要更多的栈空间且超过了可用大小时，栈会被转移到一个单独的更大的内存块。
* 改进了垃圾回收机制。Go已经在堆上实现了精准的垃圾回收，Go 1.3增加了栈上的垃圾回收。另外，GC的速度也得到了提升，现在采用的是并发清除算法，可以缩短50-70%的GC中断时间。
* 重构了链接器。对链接器和编译器进行了重构，链接器仍然是使用C语言编写，但是指令选择阶段被移入到编译器中并创建了一个新的包liblink。指令选择只会在程序包被编译时执行一次，所以这这样可以加快大幅度提升大工程的编译速度。
* 其它的一些改进。比如实现了新的正则表达式引擎、更快的race detector、默认栈的大小从8K变为4K 字节、资源竞争的检测快了40%、增加了很多新参数等。


读者可以在[这里](http://golang.org/dl/)下载Go 1.3。详细的改进说明可以阅读[官方文档](http://tip.golang.org/doc/go1.3)。不能翻墙的用户可以使用社区提供的镜像来[下载](http://golangtc.com/download)。