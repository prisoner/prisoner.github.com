---
layout: post
title: oh-my-zsh下go命令自动完成
---

新建文件$HOME/.oh-my-zsh/completions/_go, 内容如下:

{% highlight bash %}
#compdef go

_arguments "1:commands:((\
build\:'compile packages and dependencies' \
clean\:'remove object files' \
doc\:'run godoc on package sources' \
env\:'print Go environment information' \
fix\:'run go tool fix on packages' \
fmt\:'run gofmt on package sources' \
get\:'download and install packages and dependencies' \
install\:'compile and install packages and dependencies' \
list\:'list packages' \
run\:'compile and run Go program' \
test\:'test packages' \
tool\:'run specified go tool' \
version\:'print Go version' \
vet\:'run go tool vet on packages' \
))"
{% endhighlight %}

效果如下:

![自动完成效果](/images/post_images/2012-7-24-go_zsh_autocomplete.PNG)
