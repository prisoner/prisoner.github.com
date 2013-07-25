---
layout: post
title: Linux Shell 的一些使用技巧
---

Linux Shell 的一些使用技巧

* 常用快捷键
		- Ctrl + a ：移到命令行首
		- Ctrl + e ：移到命令行尾
		- Ctrl + f ：按字符前移（右向）
		- Ctrl + b ：按字符后移（左向）
		- Alt + f ：按单词前移（右向）
		- Alt + b ：按单词后移（左向）
		- Ctrl + xx：在命令行首和光标之间移动
		- Ctrl + u ：从光标处删除至命令行首
		- Ctrl + k ：从光标处删除至命令行尾
		- Ctrl + w ：从光标处删除至字首
		- Alt + d ：从光标处删除至字尾
		- Ctrl + d ：删除光标处的字符
		- Ctrl + h ：删除光标前的字符
		- Ctrl + y ：粘贴至光标后
		- Alt + c ：从光标处更改为首字母大写的单词
		- Alt + u ：从光标处更改为全部大写的单词
		- Alt + l ：从光标处更改为全部小写的单词
		- Ctrl + t ：交换光标处和之前的字符
		- Alt + t ：交换光标处和之前的单词
		- Alt + Backspace：与 Ctrl + w 类似，分隔符有些差别
		- Ctrl + l：清屏

* 使用 `Ctrl + z` 快捷键可以让正在执行的命令挂起。如果要让该进程在后台执行，那么可以执行 `bg` 命令。而 `fg` 命令则可以让该进程重新回到前台来。使用 `jobs` 命令能够查看到哪些进程在后台执行。 你也可以在 `fg` 或 `bg` 命令中使用作业 id，如： `fg %12` 又如： `bg %37`
* man手册
	* 使用书签来标记需要重复阅读的内容。方法为：先按m键，然后在 `mark:` 后输入标记字母，如 p。 当你需要返回先前设置的书签时，可以按` ' `键（单引号）。此时会显示 `goto mark:`，输入你设置的标记符即可。
	* 在阅读man手册页时想要对命令的用法进行尝试的话，那么可以使用` !`。
* `tail -f /path/to/file.log | sed '/^Finished: SUCCESS$/ q' `

    当file.log里出现Finished: SUCCESS时候就退出tail，这个命令用于实时监控并过滤log是否出现了某条记录
* `ssh user@server bash < /path/to/local/script.sh `

    在远程机器上运行一段脚本。这条命令最大的好处就是不用把脚本拷到远程机器上。
* `vim scp://username@host//path/to/somefile`

    vim一个远程文件

* `python -m SimpleHTTPServer`

    一句话实现一个HTTP服务，把当前目录设为HTTP服务目录，可以通过http://localhost:8000访问
