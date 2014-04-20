---
date: 2013-10-13
title: Mac环境下的JNI例子
---

Mac环境下的JNI例子

Java可以通过JNI接口访问本地的动态连接库, 来调用C或者C++实现的功能.
使用Java JNI主要有几个步骤:

1. 编写Java代码 注明native方法.
2. 编译Java代码得到class文件.
3. 使用javah -jni生成该类对应的C语言头文件.
4. 使用C/C++实现头文件中声明的函数.
5. 编译C/C++实现代码, 生成动态连接库.

下面是一个简单的例子

* 创建Java文件

```java
class HelloWorld {
    private native void print();
    public static void main(String[] args) {
        new HelloWorld().print();
    }
    static {
        System.loadLibrary("HelloWorld");
    }
}
```

* 编译Java代码

`javac helloWorld.java`

* 生成C语言头文件

`javah -jni HelloWorld`

* 实现头文件声明的函数

```cpp
#include <jni.h>
#include <iostream>
#include "HelloWorld.h"
using namespace std;

JNIEXPORT void JNICALL
Java_HelloWorld_print(JNIEnv *, jobject){
    cout << "hello JNI\n";
    return;
}
```

* 生成动态链接库

编译C++代码的时候在MacOS下和在Linux Windows有所不同, 不是编译成.so或者dll, 而是MacOS自己的jnilib. 并且jni.h的目录也比较特殊, 是/System/Library/Frameworks/JavaVM.framework/Headers/, 这个需要稍微注意一下.

命令:

`g++ -dynamiclib -o libhelloworld.jnilib HelloWorld.cpp -framework JavaVM -I/System/Library/Frameworks/JavaVM.framework/Headers`

一切成功后, 运行`java HelloWorld`, 就可以看到输出`hello JNI`了.

带参数和返回值的例子:

Java文件:

```java
class HelloWorld {
    private native String print(String str);
    public static void main(String[] args) {
        HelloWorld o = new HelloWorld();
        String str = o.print("JNI");
        System.out.println(str);
    }
    static {
        System.loadLibrary("HelloWorld");
    }
}
```

实现:

```cpp
#include <jni.h>
#include <iostream>
#include <cstring>
#include "HelloWorld.h"
using namespace std;

JNIEXPORT jstring JNICALL
Java_HelloWorld_print(JNIEnv *env, jobject obj, jstring str){
  const char *cstr = env->GetStringUTFChars(str, 0);
  char cap[128] = "hello ";
  strcat(cap, cstr);
  env->ReleaseStringUTFChars(str, cstr);
  return env->NewStringUTF(cap);
}
```
