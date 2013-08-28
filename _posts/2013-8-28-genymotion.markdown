---
layout: post
title: Android模拟器Genymotion
---

# Android模拟器Genymotion

Genymotion是由Genymobile开发的一款Android模拟器. 相比google的模拟器, 速度快了不只一点点. 可以说是目前世界上最快的Android模拟器.

Genymotion是基于VitualBox开发, 支持绝大部分的模拟器功能与感应器. Genymotion可运行于各个操作系统平台, 并提供Eclipse IntelliJ插件. 

## 安装

1. 注册Genymotion账号 [注册链接](http://www.genymotion.com/)
2. 安装[VitualBox](https://www.virtualbox.org/)
3. 下载安装Genymotion [下载链接](https://cloud.genymotion.com/page/launchpad/download/)
4. 安装Eclipse或者Intellij插件 (可选操作)

_注意:_ 安装完成后登录时的用户名是你注册时填写的邮件地址.  

登录界面:

![登录界面](https://cloud.genymotion.com/static/images/doc/screenshots/genymotion-connect.png)

主界面:

![主界面](https://cloud.genymotion.com/static/images/doc/screenshots/genymotion-main-window-submenu.png)

设备设置:

![设备设置](https://cloud.genymotion.com/static/images/doc/screenshots/genymotion-virtual-device-settings.png)

设备运行界面:

![设备运行界面](https://cloud.genymotion.com/static/images/doc/screenshots/genymotion-player-ready.png)

## 使用VirtualBox的共享文件夹

Genymotion是基于VitualBox开发的, 所以可以使用VitualBox的一些功能. 例如共享文件夹.
打开VirtualBox虚拟机, 选择要设置共享文件夹的Android模拟器. 进入`Settings`, 添加要共享的目录, 选中`Auto-mount选项`. 打开模拟器, 在模拟器的`/mnt/shared/`目录下已经有了我们刚才在VirtualBox 里共享的文件夹了.


