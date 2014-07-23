---
title: 使用Python的10个常见错误
date: '2014-07-23'
description:
categories:

tags:

---

从[10 Most Common Python Mistakes](http://www.toptal.com/python/top-10-mistakes-that-python-programmers-make)抄来的.

使用 [iPython](http://ipython.org/) 生成.

## #1: 错误的使用表达式做为函数默认值


    def foo(a = []):
        a.append('bar')
        return a

    foo()
    foo()
    foo()
    foo()




    ['bar', 'bar', 'bar', 'bar']



## #2: 不正确的使用类变量


    class A:
        a = 1

    class B(A):
        pass

    class C(A):
        pass

    B.a = 2
    print A.a, B.a, C.a
    A.a = 3
    print A.a, B.a, C.a

    1 2 1
    3 2 3


## #3: 错误的异常块参数


    def f():
        #a = None
        try:
            l = [1, 2, 3]
            a = l[100]
        except (ValueError, IndexError) as e:
            pass
        return a

    print f()


    ---------------------------------------------------------------------------
    UnboundLocalError                         Traceback (most recent call last)

    <ipython-input-40-ffa24284290f> in <module>()
          8     return a
          9
    ---> 10 print f()


    <ipython-input-40-ffa24284290f> in f()
          6     except (ValueError, IndexError) as e:
          7         pass
    ----> 8     return a
          9
         10 print f()


    UnboundLocalError: local variable 'a' referenced before assignment


## #4: 错误的理解Python变量作用域


    x = 100
    def foo():
        # global x
        x += 101
        print x

    foo()

## #5: 遍历list时修改了该list


    numbers = [x for x in xrange(0, 10)]
    for i in numbers:
        if bool(i % 2):
            del numbers[i]


## #6: 错误的理解closure的变量绑定


    def create_multipliers():
        return [lambda x : x * i for i in range(0, 5)]

    for i in create_multipliers():
        print i(2)


## #7: 错误的模块依赖


    #a.py
    import b

    def f():
        return b.x

    print f()

    #b.py
    import a

    x = 1

    def g():
        print a.f()


## #8: 命名容易和标准模块混淆

## #9: 未注意Python 2 和 Python 3 的差异

## #10: 错误的使用__del__方法


    import foo

    class Bar(object):
        def __del__(self):
            foo.cleanup(self.myhandle)

    #当解释器关闭时, 所有的global 变量都会被设置成None. 所以此时调用__del__会触发AttributeError错误.
