---
layout: post
title: Java 8全面解析
---

转载自:[InfoQ](http://www.infoq.com/cn/news/2013/08/everything-about-java-8)

TechEmpower是位于加利福尼亚州埃尔塞贡多的一家定制应用开发公司，该公司发表了一篇题为“[Java 8全面解析](http://www.techempower.com/blog/2013/03/26/everything-about-java-8/)”的博客文章。该博客文章全面概括了开发者在即将到来的Java 8中所要面对的变化。下面的内容快速概括了该博客文章中的信息。如果想查看所有的细节请访问TechEmpower的博客文章。

## 改进接口

现在可以在接口中定义静态方法了。例如，`java.util.Comparator`接口中现在拥有一个静态的`naturalOrder`方法。

```java
public static <T extends Comparable<? super T>> Comparator <T>naturalOrder() {
  return (Comparator<T>) Cmparators.NaturalOrderComparator.INSTANCE;
}
```

还能够在接口中提供默认方法。通过该功能，程序员能够在不破坏已有的接口实现代码的前提下添加新方法。例如，java.lang.Iterable接口现在拥有一个默认的forEach方法。

```java
public default void forEach(Consumer<? super T> action) {
  Objects.requireNonNull(action);
  for (T t : this) {
    action.accept(t);
  }
}
```

注意，接口不能为Object类中的任何方法提供默认的实现。

## 函数式接口

函数式接口是只定义了一个抽象方法的接口。Java 8引入了[FunctionalInterface](http://download.java.net/jdk8/docs/api/java/lang/FunctionalInterface.html)注解来表明一个接口打算成为一个函数式接口。例如，java.lang.Runnable就是一个函数式接口。

```java
@FunctionalInterface
public interface Runnable {
  public abstract void run();
}
```

注意，不管FunctionalInterface注解是否存在，Java编译器都会将所有满足该定义的接口看作是函数式接口。

## Lambda

函数式接口的重要属性是：我们能够使用lambda实例化它们，Lambda表达式让你能够将函数作为方法参数，或者将代码作为数据对待。下面是Lambda的一些例子。输入在左边，代码在右边。输入类型能够被推断出来，同时是可选的。

```java
(int x, int y) ->{ return x + y; }
(x, y) -> x + y
x -> x * x
() -> x
x -> { System.out.println(x); }
```

下面是实例化Runnable函数式接口的一个例子。

```java
Runnable r = () ->{ System.out.println("Running!"); }
```

## 方法引用

[方法引用](http://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)
是简洁的Lambda表达式，能够用于已经拥有名称的方法。下面是一些方法引用的例子，右边是同样效果的Lambda表达式。

```java
String::valueOf x ->String.valueOf(x)
Object::toString x ->x.toString()
x::toString () ->x.toString()
ArrayList::new () -> new ArrayList<>()
```

## 与捕获相对的非捕获Lambda

如果使用Lambda表达式访问一个在Lambda语句体外定义的非静态变量或者对象，那么它会被说成是“捕获”。
例如，下面的Lambda会访问变量x:

```java
int x = 5;
return y -> x + y;
```

一个Lambda表达式仅能够访问final或者有效final封闭块中的局部变量和参数。

## java.util.function

新版本向 [java.util.function](http://download.java.net/jdk8/docs/api/java/util/function/package-summary.html)包中添加了很多新的函数式接口。下面是一些例子：

*   Function<T, R>——将T作为输入，返回R作为输出
*   Predicate<T>——将T作为输入，返回一个布尔值作为输出
*   Consumer<T>——将T作为输入，不返回任何内容
*   Supplier<T>——没有输入，返回T
*   BinaryOperator<T>——将两个T作为输入，返回一个T作为输出

## java.util.stream

新的 [java.util.stream](http://download.java.net/jdk8/docs/api/java/util/stream/package-summary.html)包提供了对值流进行函数式操作的类。从一个集合中获取流的一种常见方式是：

```java
Stream<T> stream = collection.stream();
```

下面是一个来自于Javadocs包中的例子。

```java
intsumOfWeights = blocks.stream().filter(b ->b.getColor() == RED).mapToInt(b ->b.getWeight()).sum();
```

在该例子中我们首先使用了一个块集合作为流的来源，然后在流上执行了filter-map-reduce操作获取红块重量的和。
流可以是无限的、有状态的，可以是顺序的，也可以是并行的。
在使用流的时候，你首先需要从一些来源中获取一个流，执行一个或者多个中间操作，然后执行一个最终操作。
中间操作包括filter、map、flatMap、peel、distinct、sorted、limit和substream。
终止操作包括forEach、toArray、reduce、collect、min、max、count、anyMatch、allMatch、noneMatch、findFirst和findAny。
[java.util.stream.Collectors](http://download.java.net/jdk8/docs/api/java/util/stream/Collectors.html)是一个非常有用的实用类。
该类实现了很多归约操作，例如将流转换成集合和聚合元素。

## 改进了泛型推断

这提升了Java编译器推断泛型和在泛型方法调用中减少显式类型参数的能力。在Java 7中，代码如下：

```java
foo(Utility.<Type>bar());
Utility.<Type>foo().bar();
```

在Java 8中，改进后的参数和调用链推断让你能够按照下面的方式编写代码：

```java
foo(Utility.bar());
Utility.foo().bar();
```

## java.time

新的日期/时间API包含在 [java.time](http://download.java.net/jdk8/docs/api/java/time/package-summary.html)包中。所有的类都是不可变且线程安全的。日期和时间类型包括Instant、LocalDate、LocalTime、LocalDateTime和ZonedDateTime。除了日期和时间之外，还有Duration和Period类型。另外，值类型包括Month、DayOfWeek、Year、 Month、YearMonth、MonthDay、OffsetTime和OffsetDateTime。这些新的日期/时间类大部分JDBC都支持。

新增集合API

接口可以拥有默认函数的能力让Java 8得以向集合API中添加大量的新方法。所有的接口都提供了默认的实现，而更加有效的实现则是被添加到了具体的类中。下面是新方法的列表：

*   Iterable.forEach(Consumer)
*   Iterator.forEachRemaining(Consumer)
*   Collection.removeIf(Predicate)
*   Collection.spliterator()
*   Collection.stream()
*   Collection.parallelStream()
*   List.sort(Comparator)
*   List.replaceAll(UnaryOperator)
*   Map.forEach(BiConsumer)
*   Map.replaceAll(BiFunction)
*   Map.putIfAbsent(K, V)
*   Map.remove(Object, Object)
*   Map.replace(K, V, V)
*   Map.replace(K, V)
*   Map.computeIfAbsent(K, Function)
*   Map.computeIfPresent(K, BiFunction)
*   Map.compute(K, BiFunction)
*   Map.merge(K, V, BiFunction)
*   Map.getOrDefault(Object, V)

## 新增并发API

Java 8还向并发API中添加了一些新内容，我们将会在此简要介绍其中的一部分。
`ForkJoinPool.commonPool()`是处理所有并行流操作的结构。
没有明确提交到某个特定池中的所有`ForkJoinTask`都将会使用通用池。
`ConcurrentHashMap`已经被完全重写。
`StampedLock`是一个新的锁实现，它可以作为`ReentrantReadWriteLock`的一个备选方案。
`CompletableFuture`是`Future`接口的一个实现，它为异步任务的执行和链接提供了方法。

## 新增IO/NIO API

在Java 8中有一些新的IO/NIO方法，我们能够使用它们从文件或者输入流中获取`java.util.stream.Stream`。

*   BufferedReader.lines()
*   Files.list(Path)
*   Files.walk(Path, int, FileVisitOption...)
*   Files.walk(Path, FileVisitOption...)
*   Files.find(Path, int, BiPredicate, FileVisitOption...)
*   Files.lines(Path, Charset)
*   DirectoryStream.stream()

这里面有一个新的`UncheckedIOException`，它是一个继承了`RuntimetimeException`的`IOException`。
还有一个`CloseableStream`，它是一个能够并且应该被关闭的流。

## 反射和注解的变化

通过[类型注解](http://types.cs.washington.edu/jsr308/)，我们能够在更多的地方使用注解，
例如像`List<@Nullable String>`这样的泛型参数中。
这增强了通过静态分析工具发现错误的能力，它将增强并重定义Java内置的类型系统。

## Nashorn JavaScript引擎

Nashorn是一个集成到JDK中的新的、轻量级、高性能的JavaScript实现。
Nashorn是Rhino的继任者，它提升了性能和内存使用情况。
它将会支持`javax.script API`，但是它并不会支持DOM/CSS，也不会包含浏览器插件API。

## java.lang、java.util等其他地方的新增功能

Java 8还向很多其他的包中添加了大量其他的功能，在本文中我们并没有提及。
下面是一些值得注意的内容。可以使用ThreadLocal.withInitial(Supplier)更加简洁的声明本地线程变量。
长期未兑现的StringJoiner和String.join(...)现在已经是Java 8的一部分了。
比较器提供了一些新的方法能够用于链接和基于域的比较。默认的字符串池映射大小更大了，大约在25—50K。

如果想要获取更加详细的介绍可以访问博客文章[Java 8全面解析](http://www.techempower.com/blog/2013/03/26/everything-about-java-8/)。该博客文章的最后一次更新时间是2013年5月29日。

**查看英文原文**：[Everything About Java 8](http://www.infoq.com/news/2013/08/everything-about-java-8)
