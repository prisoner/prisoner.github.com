---
date: 2013-09-13
title: Vagrant中的VirtualBox配置
---

VirtualBox provider暴露了一些额外的配置项可以用于配置VirtualBox选项.

## 配置虚拟机名称

```ruby
config.vm.provider "virtualbox" do |v|
  v.name = "my_vm"
end
```

## 配置是否使用headless模式

Vagrant默认使用VirtualBox的headless模式. 如果需要也可以设置不使用.

```ruby
config.vm.provider "virtualbox" do |v|
  v.gui = true
end
```

## VBoxManage设置

Vagrant可以使用[VBoxManage](http://www.virtualbox.org/manual/ch08.html)进行一些额外的配置.

```ruby
config.vm.provider "virtualbox" do |v|
  v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
end
```

* 配置中的`:id`会在创建虚拟机时使用实际的虚拟机ID替换.
* 数组中的所有参数会被组合做为VBoxManage的参数传入.
* 如果有多个`customize`, 会被按顺序执行.

详细的可供修改参数请参考[VBoxManage](http://www.virtualbox.org/manual/ch08.html), 以下是一些常用参数.

**modifyvm 使用的参数:**

1. --ostype <ostype>: 指定虚拟机的类型. 类型可以使用`VBoxManage list ostypes`查看.
1. --memory <memorysize>: 指定虚拟机使用的内存, 单位是MB.
1. --vram <vramsize>: 指定显卡使用的内存.
1. --cpus <cpucount>: 指定使用cpu的个数.
1. --rtcuseutc on|off: 指定硬件时钟使用UTC.
1. --cpuexecutioncap <1-100>: 设置虚拟机使用cpu的运行峰值.
1. --boot<1-4> none|floppy|dvd|disk|net: 指定启动设备.
1. --snapshotfolder default|<path>: 指定snapshot保存目录.
