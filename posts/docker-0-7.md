---
date: 2013-11-27
title: Docker发布0.7版本
---

轻量级容器工具Docker发布0.7版本. 新版本最大的亮点是带来了标准Linux支持, 不再需要特定的内核.

新版本带来七个特性:

1. 标准Linux支持, 提供对Fedora, RHEL, Ubuntu, Debian, Suse, Gentoo, Arch, 等发行版的支持. 还是没有CentOS...
2. 存储驱动. 之前的版本一直使用AUFS做为存储, 但是AUFS并不在Linux内核中. 所以只有少数发行版能直接支持Docker. 新的storage driver API改变了这一现状, 实现统一的存储驱动, 包含了对三种存储方式的支持: AUFS, VFS和DeviceMapper 的支持. 正在开发包括Btrfs和ZFS.
3. 离线传输. 使用离线传输, container images可以打包迁移到其它运行机器.
4. 改善的端口映射. 支持更复杂的端口映射. 同时调整了端口映射语法.
5. 容器连接. 允许各个容器进行通信. 详细:[Working with Links and Names](http://docs.docker.io/en/latest/use/working_with_links_names/#links-service-discovery-for-docker)
6. 容器自定义名称.
7. 质量改善.

详细请参考[官方发布公告](http://blog.docker.io/2013/11/docker-0-7-docker-now-runs-on-any-linux-distribution/)
