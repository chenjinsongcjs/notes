# 初始Go语言

### 1.1 Go的语言特性：

-   开发源码 ：go语言本身就是go语言编写的
-   静态类型和编译类型 ：属于强类型语言
-   跨平台 ：支持多种计算架构的操作系统
-   自动垃圾回收
-   原生的并发编程 ：goroutine、channel和go关键字
-   支持函数式编程 ：函数是一等公民
-   代码风格强制一致 ：gofmt在编译时强制格式化
-   高效的编程和运行
-   丰富的标准库

### 1.2 Go语言的安装和设置

-   官方网址：[https://golang.org](https://golang.org "https://golang.org")
-   国内代理：[https://goproxy.cn](https://goproxy.cn "https://goproxy.cn")
-   设置GOROOT和GOPATH（windows会自动设置）

### 1.3  Go安装包中的相关文件

-   api 文件夹：用于存储Go版本顺序的API增量列表文件
-   bin文件夹 ：用于存储主要的标准命令文件，包括go、gofmt等
-   doc文件夹：用于存储标准库的HTML格式的程序文档
-   lib文件夹： 存储一些特殊的库文件
-   misc文件夹：用于存储一些辅助类的说明和工具
-   pkg文件夹：用于存储安装go标准库后的所有归档文件，主要是按照操作系统+计算机架构命名如：linux\_amd64
-   src文件夹：用于存放Go自身、Go标准工具以及标准库的所有源码文件
-   test文件夹 ：存放用来测试和验证Go本身的所有相关文件

### 1.4 Go的工作区

-   go的工作区一般就是GOPATH的路径，一般包含三个子目录
-   src目录：主要是存储Go的源码文件
-   pkg目录：用于存储go install 命令安装后的代码包的归档文件，一般是以".a"结尾的文件
-   bin目录：存放go install 命令完成后安装的可执行文件，在linux中和源码文件相同，在window中会有".exe"后缀

### 1.5 GOPATH、vendor、gomod模式

-   GOPATH：每个项目都会有自己的GOPATH或者使用全局的GOPATH。通过go get下载的第三方依赖会被存储在GOPATH中。
-   vendor：每一个项目都会有自己的vendor目录，主要是用于解决不同项目对同一个依赖版本之间的问题
-   gomod：现在的主流模式，使用简单，只需要一个go.mod文件就能解决上面两种方式的问题

### 1.6  代码包

-   在Go中代码包是代码编译和安装的基本单元
-   包的声明必须在源码文件非注释的第一行，包名可以任意命名
-   一个包中可以包含多个源码文件
-   含有main函数的包一定为main包
-   可以通过import语句导入当前程序依赖的包，在go.mod模式下可以直接导入第三方包，会自动下载
-   在导入包的时候为了避免包的命名冲突，可以为包起别名
-   可以使用 “.”完全导入包，在引用包的函数时，不同写包名
-   可以使用 “ \_”初始化包但是不使用包中的任何函数

```go
import (
    .  "fmt" 
    _ "io/ioutil"
    http  "net/http"
)
```

-   包的初始化：
    -   在main函数执行之前会导入所有的包，依次初始化每个包的变量，然后依次执行每个包的init函数返回

### 1.7 标准命令的简述&#xD;

-   go build :用于编译执行的源码文件，含有main函数的源码文件会被编译为可执行文件
-   go clean ：用于清除执行其他go命令留下来的目录或者文件
-   go doc :用于显示指定函数或者包的注释文档
-   go env :用于答应Go语言的相关环境信息
-   go fix :用于修正指定代码包源码文件中的过时语法和代码调用
-   go fmt: 用于格式化源码文件，实际上是通过gofmt做的
-   go get ：用于下载、编译安装指定的代码包及其依赖
-   go install :用于编译安装指定的代码包及其依赖
-   go list:用于显示指定代码包的信息，配合text/template使用，进行灵活输出
-   go run：用于编译执行指定的源码文件
-   go test:用于执行指定的测试源码文件，一般为“xxx\_test.go”文件
-   go tool:用于执行Go的特殊工具
-   go vet :用于Go语言源码
-   go version:查看版本

### 1.8 一些通用的命令参数

-   \-a 强行重新编译所涉及到的所有Go语言代码包，包括标准库的包，可以用于改动源码做实验
-   \-n 打印命令执行过程中用到的所有命令，并且不会执行他们
-   \-race 检测并报告Go语言中存在的数据竞争问题，是并发程序中重要的检测手段
-   \-v 打印命令执行过程中所涉及的包
-   \-work 打印命令执行过程中使用到的临时工作目录，并且不会删除他们，如果没有使用这个参数就会删除这些临时目录
-   \-x 打印命令执行过程中用到的所有命令，并且会执行他们

### 1.9 常用工具

-   pprof ：以交互式的方式访问一些性能概要的文件。包括一些cpu、内存和程序阻塞的文件
    -   e.g : go tool pprof cpu.out&#x20;
    -   go test -bench . -cpuprofile cpu.out 获取cpu的性能文件
-   trace :用于读取go程序的踪迹文件，并以图形化的形式展现出来，比如，堆的使用情况，gorutine是如何调用的等等
-   这两个都得到了go test的支持

### 1.10 实例代码

```go
package main
import (
    "bufio"
    "fmt"
    "os"
)
func main() {
    reader := bufio.NewReader(os.Stdin)
    fmt.Println("Please input a word :")
    input, err := reader.ReadString('\n')
    if err != nil {
        panic(err)
    }
    fmt.Print(input)
}

```
