---
layout: post
title: Linux 文件 Loop设备 LVM
---

不知道标题该怎么取. 本文主要介绍在Linux下怎样将文件挂载成磁盘并且用LVM进行分区管理.

## 什么是loop设备

loop设备是一种伪设备. 是使用文件来模拟块设备的一种技术. 文件模拟成块设备后, 就像一个磁盘或光盘一样使用. 回环可以理解成回复重用, 在已有设备上建立文件来模拟物理块设备.

## 关联loop设备

一般在linux中会有8个loop设备. 我们可以将文件关联到这些设备上.
查看所有的loop设备: `losetup -a`, 输出:
```
/dev/loop0: [0806]:5373954 (/mnt/var/disk/disk0.img)
/dev/loop1: [0806]:5373955 (/mnt/var/disk/disk1.img)
/dev/loop2: [0806]:5373956 (/mnt/var/disk/disk2.img)
/dev/loop3: [0806]:5373957 (/mnt/var/disk/disk3.img)
```

查看下一个未使用的loop设备: `losetup -f`, 输出:
```
/dev/loop4
```

使用`dd`或者`truncate`创建文件. 如:
`truncate -s 10M disk0.img`
或者
`dd if=/dev/zero of=disk0.img ibs=1M count=10`

然后执行:
`losetup /dev/loop0 disk0.img`
将文件disk0.img关联到`/dev/loop0`.

这时候disk0.img已经可以做为块设备使用了. 我们可以将它分区,并创建文件系统, 然后挂载, 就好像使用一个普通的硬盘一样.

## 使用LVM管理分区

我们也可以使用LVM来管理分区.

按照上文创建3个loop设备.
```
/dev/loop0
/dev/loop1
/dev/loop2
```
### 创建物理卷

```
pvcreate /dev/loop0 /dev/loop1 /dev/loop2
```

### 创建卷组

```
vgcreate vg0 /dev/loop0 /dev/loop1 /dev/loop2
```

### 创建逻辑卷

```
lvcreate --size 20m --name lv0 vg0
```

我们现在已经有一个使用3个文件来创建的lvm逻辑卷了. 对这个逻辑卷创建文件系统并挂载后就可以像普通文件系统一样使用了.
如果在以后的使用中觉得空间不够的话, 还可以创建新的文件并关联loop设备, 然后添加到lvm的卷组中, 并且对逻辑卷进行在线扩容. 具体可以参考LVM的使用手册.

