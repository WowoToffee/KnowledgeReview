### 使用go path问题

1. 代码开发必须在go path src目录下，不然，就有问题。
2. 依赖手动管理
3. 依赖包没有版本可言

从这个看， go path不算包管理工具

### go mod介绍

go modules 是 golang 1.11 新加的特性。Modules官方定义为：

> 模块是相关Go包的集合。modules是源代码交换和版本控制的单元。 go命令直接支持使用modules，包括记录和解析对其他模块的依赖性。modules替换旧的基于GOPATH的方法来指定在给定构建中使用哪些源文件。

### 如何使用go mod

下面设置go mod和go proxy

```bash
go env -w GOBIN=/Users/youdi/go/bin
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct // 使用七牛云的
```

### GO111MODULE

GO111MODULE 有三个值：off, on和auto（默认值）。

GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
 GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
 GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。这种情况下可以分为两种情形：

### go mod命令

go mod 有以下命令：

| 命令     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| download | download modules to local cache(下载依赖包)                  |
| edit     | edit go.mod from tools or scripts（编辑go.mod)               |
| graph    | print module requirement graph (打印模块依赖图)              |
| verify   | initialize new module in current directory（在当前目录初始化mod） |
| tidy     | add missing and remove unused modules(拉取缺少的模块，移除不用的模块) |
| vendor   | make vendored copy of dependencies(将依赖复制到vendor下)     |
| verify   | verify dependencies have expected content (验证依赖是否正确） |
| why      | explain why packages or modules are needed(解释为什么需要依赖) |



### 使用go mod管理一个新项目

```sh
mkdir Gone
cd Gone
go mod init Gone

go get github.com/gin-gonic/gin@v1.6.3 // 不加版本号默认最新的版本（如果需要修改版本时直接修改版本号然后再次执行就好了）


```



