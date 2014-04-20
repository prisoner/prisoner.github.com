---
date: 2013-09-17
title: 使用Supervisor管理进程
---

Supervisor是用来管理和监控进程的工具. 有时候我们希望我们自己开发的程序也能做到随系统自动启动, 而且启动之后最好还能方便的控制其停止/重启.  并且在出现问题的时候能够自动重启.

一般我们可以在`/etc/init.d/`下建立启动脚本, 但是启动脚本写起来比较繁琐, 并且不能自动重启.
这时候我们就可以使用Supervisor了.

## 安装

ubuntu系统下:

```
sudo apt-get install supervisor
```

## 配置

配置文件默认为`/etc/supervisord.conf`. 如果要添加一个被管理的进程, 在该文件加入配置:

```
[program:test]
command=python path/to/test.py
autostart=true
autorestar=unexpected
stdout_logfile=path/to/log.log
```

* [program:test] test是指定控制任务时使用的名称.
* command=python path/to/test.py 被控制的程序, 可以是任何命令
* autostart=true 是否随着supervisord启动而启动
* autorestar=unexpected 当进程死亡后, 是否重新启动进程
* stdout_logfile=path/to/log.log stdout输出的日子文件

配置文件更改后, 需要执行

```
sudo supervisorctl update
```
来更新配置

更多详细配置可以参考[官方文档](http://supervisord.org/configuration.html#program-x-section-values)

## 使用

Supervisor安装后有两个可执行命令:

* supervisord 后台守护进程
* supervisorctl 控制程序, 用来向守护进程发送命令

supervisorctl命令:

1. supervisorctl status test 查看test的状态
1. supervisorctl start test 启动test
1. supervisorctl stop test 停止test

更多命令请参考: `supervisorctl help`

## web客户端

在配置中有一段`[inet_http_server]`,  配置好后就打开`127.0.0.1:9001`, 就可以看到一个网页版控制界面了.

配置:

```
[inet_http_server]
port = 127.0.0.1:9001
username = user
password = 123
```
