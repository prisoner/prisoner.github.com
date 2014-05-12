---
date: 2013-08-30
title: vagrant介绍
---

# vagrant介绍

Vagrant是一个使用ruby编写基于VirtualBox的, 用于创建和配置轻量级的 可重用的 便携的虚拟化开发环境的工具.

可以满足以下需求:

* 创建隔离依赖的开发环境
* 创建可重用的干净的测试环境
* 创建可重用的工作流程

## 安装

1. 安装Ruby, VirtualBox
2. 下载安装[Vagrant](http://downloads.vagrantup.com/)

## 配置

Vagrant默认文件会保存在用户目录下. 如过要改动, 可以移动`~/.vagrant.d`到你想要位置. 并且设置环境变量`VAGRANT_HOME`指向新路径.   
同时可以修改Vagrant创建的虚拟机的保存位置. 打开VirtualBox程序, 点击`管理/全局设定`菜单项, 将`常规`栏里的`默认虚拟电脑位置(M)`改为其他路径.

## 使用

### Box

box就相当于是一个环境，它一般是一个VirtualBox虚拟机的镜像，官方提供了一个基于Ubuntu 10.04的box。 给vagrant添加一个box:

    vagrant box add lucid32 http://files.vagrantup.com/lucid32.box

其中lucid32表示给这个box起名lucid32。

也可以把lucid32.box下载过来，然后直接执行

    vagrant box add lucid32 lucid32.box

[这个网站](http://www.vagrantbox.es/)收集了很多用户自己创建的box, 可以根据需要选择下载.

### 创建虚拟机

执行

    vagrant init lucid32
    
表示基于lucid32创建一个虚拟环境, 命令执行后会在当前目录下生成一个Vagrantfile文件. 这个就是当前虚拟环境的配置文件.

再执行

    vagrant up

就可以启动虚拟机了.  默认启动后是没有界面的. 

执行

    vagrant ssh

就可以ssh到虚拟机了.

Vagrantfile有几项比较重要的配置:

* config.vm.box 虚拟机使用的Box
* config.vm.network 配置虚拟机网络连接方式
* config.vm.forward_port 配置虚拟机端口转发
* config.vm.share_folder 配置文件夹共享目录

### 打包Box

对虚拟机做过一些配置后, 可以对虚拟机的当前状态进行打包, 已供其他人使用.

执行`vagrant package`即可在当前目录生成一个package.box文件.

### 其它命令

* vagrant halt 关闭虚拟机
* vagrant suspend 休眠虚拟机
* vagrant reload 重启虚拟机
* vagrant status 查看当前虚拟机状态
* 更多其他命令可以通过vagrant help查看。

## 参考

* [Vagrant docs](http://docs.vagrantup.com/v2/)
* [Vagrant 知识澄清与杂症诊治](http://wushaobo.info/?p=83)
