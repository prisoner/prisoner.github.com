---
date: 2013-09-21
title: Go 1.2 Release Notes
---

[Go 1.2 Release Notes](http://tip.golang.org/doc/go1.2)

### nil的使用

在Go 1.0中, 以下代码

```go
type T struct {
    X [1<<24]byte
    Field int32
}

func main() {
    var x *T
    ...
}
```
`x.Field`将会访问`1<<24`指向的内存地址. 但是在Go 1.2中将会抛出`run-time panic`

### 允许设置slice容量的新切片语法

```go
var array [10]int
slice = array[2:4:6]
```
slice的容量为6.

### godoc和vet移动到go.tools

### gccgo的状态

将来GCC4.9能包含完整的Go1.2. 目前的GCC4.8.2包含Go1.1.2.

### cgo的变化

支持C++语法, 但是只支持C的导入符号. 详情参[http://tip.golang.org/cmd/cgo/](http://tip.golang.org/cmd/cgo/)

### 支持测试覆盖率

通过安装`go get code.google.com/p/go.tools/cmd/cover`, 可以获得一个支持测试覆盖率的工具.
使用:
```shell
$ go test -cover fmt
ok  	fmt	0.060s	coverage: 91.4% of statements
```

### 删除go doc命令

godoc依然可以使用

### go命令的变化

`go get`增加`-t`参数, 用于下载包的测试用例.

### 性能优化

* compress/bzip2: 30%的性能提升
* crypto/des: 5倍的性能提升
* encoding/json: encoding 30% 的性能提升
* windows和BSD下, 网络和runtime的深度集成, 30% 的性能提升.

### 标准库的变化

* encoding: 新包, 提供通用的encoding接口
* fmt: 引入参数索引支持

参考资料:

* [Go1.2新功能预览](http://my.oschina.net/chai2010/blog/160143)
* [翻译Go tip带来的变化](http://mikespook.com/2013/08/%E7%BF%BB%E8%AF%91go-tip%EF%BC%882013-08-23%EF%BC%89%E5%B8%A6%E6%9D%A5%E7%9A%84%E5%8F%98%E5%8C%96/)
