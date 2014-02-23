---
layout: post
title: Dockerfile最佳实践
---

Dockerfile提供了一个简单的语法来构建Image. 以下是几点技巧帮助你更好的使用Dockerfile.

## 使用缓存

Dockerfile中的每一个操作都会提交一个更改到一个新的Image中, 并且这个Image会做为下一个操作的Base Image. 如果一个使用相同操作的Image存在, 那么这些操作不会执行, 而是直接使用这个Image.

例如所有的Dockerfile都可以使用以下的共通代码.

```
FROM ubuntu
MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y
```

_保持通用操作在Dockerfile顶部, 以使用缓存_

## 使用标签

除非仅仅是出于实验目的, 不然你应该使用`docker build -t`命令来创建Image.

_总是使用参数`-t`_

## 暴露端口

docker的两个重要特性是可重复利用和可移植. Image应该在任何主机上能运行多次. 所以你不应该在Dockerfile中指定公开端口. 如果Image的使用者需要指定公开端口, 他/她们会使用`-p`参数的.

_不要在Dockerfile中指定公开端口_

## CMD和ENTRYPOINT语法

CMD和ENTRYPOINT都有两种语法.

```
CMD /bin/echo
# or
CMD ["/bin/echo"]
```

第一种语法在执行的时候, docker会将命令包装成`/bin/sh -c`的形式. 这在有些情况下会产生一些难以发现的错误.

_CMD和ENTRYPOINT总是使用数组形式的语法_

## CMD和ENTRYPOINT更好的配合

ENTRYPOINT可以让你docker化的程序像一个可执行程序一样运行. 你可以传递参数到ENTRYPOINT指定的命令中, 而不用担心在运行`docker run`的时候会被覆盖. 有时ENTRYPOINT和CMD同时使用会有更好的效果. 例如[Rethinkdb](http://www.rethinkdb.com/)的Dockerfile.

```
# Dockerfile for Rethinkdb
# http://www.rethinkdb.com/

FROM ubuntu

MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python-software-properties
RUN add-apt-repository ppa:rethinkdb/ppa
RUN apt-get update
RUN apt-get install -y rethinkdb

# Rethinkdb process
EXPOSE 28015
# Rethinkdb admin console
EXPOSE 8080

# Create the /rethinkdb_data dir structure
RUN /usr/bin/rethinkdb create

ENTRYPOINT ["/usr/bin/rethinkdb"]

CMD ["--help"]
```

所有运行`docker run`时传递的参数都会被传递到ENTRYPOINT指定的命令`/usr/bin/rethinkdb`中, 当没有指定参数时使用CMD指定的`--help`参数.

运行`docker run crosbymichael/rethinkdb`, 会输出:

```
Running 'rethinkdb' will create a new data directory or use an existing one,
  and serve as a RethinkDB cluster node.
File path options:
  -d [ --directory ] path           specify directory to store data and metadata
  --io-threads n                    how many simultaneous I/O operations can happen
                                    at the same time

Machine name options:
  -n [ --machine-name ] arg         the name for this machine (as will appear in
                                    the metadata).  If not specified, it will be
                                    randomly chosen from a short list of names.

Network options:
  --bind {all | addr}               add the address of a local interface to listen
                                    on when accepting connections; loopback
                                    addresses are enabled by default
  --cluster-port port               port for receiving connections from other nodes
  --driver-port port                port for rethinkdb protocol client drivers
  -o [ --port-offset ] offset       all ports used locally will have this value
                                    added
  -j [ --join ] host:port           host and port of a rethinkdb node to connect to
  .................
```

而运行`docker run crosbymichael/rethinkdb --bind all`, 会输出:

```
info: Running rethinkdb 1.7.1-0ubuntu1~precise (GCC 4.6.3)...
info: Running on Linux 3.2.0-45-virtual x86_64
info: Loading data from directory /rethinkdb_data
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
warn: Could not turn off filesystem caching for database file: "/rethinkdb_data/auth_metadata" (Is the file located on a filesystem that doesn't support direct I/O (e.g. some encrypted or journaled file systems)?) This can cause performance problems.
info: Listening for intracluster connections on port 29015
info: Listening for client driver connections on port 28015
info: Listening for administrative HTTP connections on port 8080
info: Listening on addresses: 127.0.0.1, 172.16.42.13
info: Server ready
info: Someone asked for the nonwhitelisted file /js/handlebars.runtime-1.0.0.beta.6.js, if this should be accessible add it to the whitelist.
```

_更好的配合使用CMD和ENTRYPOINT_

-----

原文: [Dockerfile Best Practices](http://crosbymichael.com/dockerfile-best-practices.html)

参考:

[How to Use Entrypoint in Docker Builder](http://www.kstaken.com/blog/2013/07/06/how-to-use-entrypoint-in-a-dockerfile/)

[Dockerfile tutorial Level1](http://www.docker.io/learn/dockerfile/level1/)

[Dockerfile tutorial Level2](http://www.docker.io/learn/dockerfile/level2/)

