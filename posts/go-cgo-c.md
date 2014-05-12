---
date: 2014-03-31
title: Golang调用C
---

有时候我们需要在go中调用一些使用C或者C++编写的代码. 而这些代码大多数会被编译成动态链接库的形式存在.

本文就以Mac OS下的Go开发为例, 来测试在GO中调用C.

首先做一个简单的动态链接库.

hello.h

```c
#ifndef HELLO_H
#define HELLO_H

void hello(const char *name);

#endif
```

hello.c

```c
#include <stdio.h>

void hello(const char *name) {
  printf("Hello %s!\n", name);
}
```

然后编译并生成动态库:

```bash
gcc -c hello.c
cc -dynamiclib -o libhello.dylib hello.o
```

编写GO代码:

```golang
package hello

/*
#cgo CFLAGS: -I../c/hello
#cgo LDFLAGS: -L../c/hello -lhello
#include <hello.h>
*/
import "C"

func Hello(str string) {
	C.hello(C.CString(str))
}
```

`CFLAGS: -I../c/hello`指定头文件的位置(hello.h)
`LDFLAGS: -L../c/hello -lhello` 指定动态库的位置(libhello.dylib).

编译并安装:

```bash
go build hello
go install hello
```

在写一个测试程序来调用:

```golang
package main

import "hello"

func main(){
    hello.Hello("world")
}
```

运行命令, 将动态库安装到系统能识别的位置.

```bash
mv libhello.dylib /usr/local/lib/
```

运行输出:

```
Hello world!
```