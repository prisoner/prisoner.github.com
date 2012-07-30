---
layout: post
title: linux 命令行下求文件的交集和差集
---

linux 命令行下求文件的交集和差集.

file1:

    a
    b
    c
    d

file2

    c
    d
    e
    f

### 方法1
* 交集 `sort file1.txt file2.txt | uniq -d`
* 差集 取在file1中但不在file2中的 `sort file2.txt file2.txt file1.txt | uniq -u`

### 方法2
* 交集 `grep -F -v -f file1.txt file2.txt`
* 差集 取在file1中但不在file2中的 `grep -F -v -f file2.txt file1.txt`

