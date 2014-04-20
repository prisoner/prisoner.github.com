---
date: 2011-07-29
title: 甲骨文发布Java SE 7正式版
---

7月29日, Java SE 7正式发布! 经过与世界范围内的Java社区,
Java平台历时将近五年的协作, 标准版本已经可以
[下载](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
了.

新特性
------

-   改进动态语言支持
-   改进的IO, 加强对NIO的支持 详见 [Enhancements in Java
    I/O](http://download.oracle.com/javase/7/docs/technotes/guides/io/enhancements.html#7)
-   支持非Java语言: Java SE 7 引入一个新的 JVM
    指令用于简化实现动态类型编程语言
-   Garbage-First Collector 新的针对服务器的垃圾收集器用于替换CMS. [What
    is the Garbage-First
    Collector?](http://download.oracle.com/javase/7/docs/technotes/guides/vm/G1.html)
-   基于ForkJoinPool类的fork/join框架.
    适合于多处理器环境下用来进行高效的运行大量任务. 详见 [Concurrency
    Utilities Enhancements in Java SE
    7](http://download.oracle.com/javase/7/docs/technotes/guides/concurrency/changes7.html)
-   增强国际化支持. [Internationalization Enhancements in Java SE
    7](http://download.oracle.com/javase/7/docs/technotes/guides/intl/enhancements.7.html)

语言特性
--------

### 二进制数字表达方式

在java7中, 整形(byte, short, int, and long)可以用二进制表示, 加上前缀 0b
或者 0B 就可以了.

<script src="https://gist.github.com/1113405.js">

</script>
### switch支持字符串变量

这可是期盼多年的特性. 表达式比较使用 `String.equals` 方法

<script src="https://gist.github.com/1113411.js">

</script>
### try-with-resources 语句

在资源使用完后自动关闭. 资源是指实现了
[java.lang.AutoCloseable](http://download.oracle.com/javase/7/docs/api/java/lang/AutoCloseable.html)
或者
[Closeable](http://download.oracle.com/javase/7/docs/api/java/io/Closeable.html)
接口的类.\
注意在try-with-resources中依然可以使用 `catch` 和 `finally`, `catch` 和
`finally` 会在资源关闭之后运行.

<script src="https://gist.github.com/1113421.js">

</script>
### 同时捕获多个异常

```
catch (IOException|SQLException ex) { logger.log(ex); throw ex; }
```

### 数字表达中使用下划线

<script src="https://gist.github.com/1113435.js">

</script>
#### 泛型在创建实例时的类型引用

之前的代码
`Map<String, List<String>> myMap = new HashMap<String, List<String>>();`
Java7的代码
`Map<String, List<String>> myMap = new HashMap<>();`
编译器可以根据声明自动推断范型的类型.

### Improved Compiler Warnings and Errors When Using Non-Reifiable Formal Parameters with Varargs Methods

详细:
[链接](http://download.oracle.com/javase/7/docs/technotes/guides/language/non-reifiable-varargs.html)

-----

可惜那个很酷的集合声明语法没有实现
`List<String> people = {"Frank", "Mary", "Satan"};`
`Map<String, Integer> map = {"key" : 1};`
是不是有些动态语言的味道

闭包也没有, 可惜…
