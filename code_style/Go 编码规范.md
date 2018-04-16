# Go 编码规范



## 代码风格

### 文件

使用`UTF-8`编码。

#### 缩进

4个空格

#### 行长约定

一行最长不超过80个字符，超过的请使用换行展示，尽量保持格式优雅。

#### go vet

vet工具可以帮我们静态分析我们的源码存在的各种问题，例如多余的代码，提前return的逻辑，struct的tag是否符合标准等。

```
go get golang.org/x/tools/cmd/vet

```

使用如下：

```
go vet .
```



### 项目目录结构规范

项目的目录结构尽量做到简介分明、层次清晰。



### 命名规范

#### 文件名

文件名使用小写，顾名思义。

#### 接口名

单个函数的接口名以”er”作为后缀，如Reader,Writer

接口的实现则去掉“er”

```
type Reader interface {
        Read(p []byte) (n int, err error)
}

```

两个函数的接口名综合两个函数名

```
type WriteFlusher interface {
    Write([]byte) (int, error)
    Flush() error
}

```

三个以上函数的接口名，类似于结构体名

```
type Car interface {
    Start([]byte) 
    Stop() error
    Recover()
}
```

#### 变量名

 `变量` 使用 `Camel命名法`。

 `常量` 使用 `全部字母大写，单词间下划线分隔` 的命名方式。

 `函数` 使用 `Camel命名法`。

 函数的 `参数` 使用 `Camel命名法`。

### package名字

保持package的名字和目录保持一致，尽量采取有意义的包名，简短，有意义，尽量和标准库不要冲突。

### import 规范

在一个文件里面引入了一个package，建议采用如下格式：

```
import (
    "fmt"
)

```

如果你的包引入了三种类型的包，标准库包，程序内部包，第三方包，建议采用如下方式进行组织你的包：

```
import (
    "encoding/json"
    "strings"

    "myproject/models"
    "myproject/controller"
    "git.obc.im/obc/utils"

    "git.obc.im/dep/beego"
    "git.obc.im/dep/mysql"
)  

```

在项目中不要使用相对路径引入包：

```
// 这是不好的导入
import “../net”

// 这是正确的做法
import “github.com/repo/proj/src/net”
```



### 错误处理

error作为函数的值返回,必须尽快对error进行处理
采用独立的错误流进行处理
不要采用这种方式

```
    if err != nil {
        // error handling
    } else {
        // normal code
    }

```

而要采用下面的方式

```
    if err != nil {
        // error handling
        return // or continue, etc.
    }
    // normal code
```

### Panic

在逻辑处理中禁用panic
在main包中只有当实在不可运行的情况采用panic，例如文件无法打开，数据库无法连接导致程序无法 正常运行，但是对于其他的package对外的接口不能有panic，只能在包内采用。 建议在main包中使用log.Fatal来记录错误，这样就可以由log来结束程序。

### Recover

recover用于捕获runtime的异常，禁止滥用recover，在开发测试阶段尽量不要用recover，recover一般放在你认为会有不可预期的异常的地方。

```
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    // do 函数可能会有不可预期的异常
    do(work)
}

```

### Defer

defer在函数return之前执行，对于一些资源的回收用defer是好的，但也禁止滥用defer，defer是需要消耗性能的,所以频繁调用的函数尽量不要使用defer。

```
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

### 控制结构

#### if

if接受初始化语句，约定如下方式建立局部变量

```
if err := file.Chmod(0664); err != nil {
    return err
}

```

#### for

采用短声明建立局部变量

```
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}

```

#### range

如果只需要第一项（key），就丢弃第二个：

```
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}

```

如果只需要第二项，则把第一项置为下划线

```
sum := 0
for _, value := range array {
    sum += value
}

```

#### return

尽早return：一旦有错误发生，马上返回

```
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

### 方法的接收器

名称 一般采用strcut的第一个字母且为小写，而不是this，me或者self

```
    type T struct{} 
    func (p *T)Get(){}

```

如果接收者是map,slice或者chan，不要用指针传递

```
//Map
package main

import (
    "fmt"
)

type mp map[string]string

func (m mp) Set(k, v string) {
    m[k] = v
}

func main() {
    m := make(mp)
    m.Set("k", "v")
    fmt.Println(m)
}

```

```
//Channel
package main

import (
    "fmt"
)

type ch chan interface{}

func (c ch) Push(i interface{}) {
    c <- i
}

func (c ch) Pop() interface{} {
    return <-c
}

func main() {
    c := make(ch, 1)
    c.Push("i")
    fmt.Println(c.Pop())
}

```

如果需要对slice进行修改，通过返回值的方式重新赋值

```
//Slice
package main

import (
    "fmt"
)

type slice []byte

func main() {
    s := make(slice, 0)
    s = s.addOne(42)
    fmt.Println(s)
}

func (s slice) addOne(b byte) []byte {
    return append(s, b)
}

```

如果接收者是含有sync.Mutex或者类似同步字段的结构体，必须使用指针传递避免复制

```
package main

import (
    "sync"
)

type T struct {
    m sync.Mutex
}

func (t *T) lock() {
    t.m.Lock()
}

/*
Wrong !!!
func (t T) lock() {
    t.m.Lock()
}
*/

func main() {
    t := new(T)
    t.lock()
}

```

如果接收者是大的结构体或者数组，使用指针传递会更有效率。

```
package main

import (
    "fmt"
)

type T struct {
    data [1024]byte
}

func (t *T) Get() byte {
    return t.data[0]
}

func main() {
    t := new(T)
    fmt.Println(t.Get())
}
```



### bug注释

针对代码中出现的bug，可以采用如下教程使用特殊的注释，在godocs可以做到注释高亮：

```
// BUG(astaxie):This divides by zero. 
var i float = 1/0

```

[http://blog.golang.org/2011/03/godoc­documenting­go­code.html](http://blog.golang.org/2011/03/godoc%C2%ADdocumenting%C2%ADgo%C2%ADcode.html)



### struct规范

#### struct申明和初始化格式采用多行：

定义如下：

```
type User struct{
    Username  string
    Email     string
}

```

初始化如下：

```
u := User{
    Username: "astaxie",
    Email:    "astaxie@gmail.com",
}

```

#### recieved是值类型还是指针类型

到底是采用值类型还是指针类型主要参考如下原则：

```
func(w Win) Tally(playerPlayer)int    //w不会有任何改变 
func(w *Win) Tally(playerPlayer)int    //w会改变数据

```

更多的请参考：<https://code.google.com/p/go-wiki/wiki/CodeReviewComments#Receiver_Type>

#### 带mutex的struct必须是指针receivers

如果你定义的struct中带有mutex,那么你的receivers必须是指针



## References

* [Golang代码规范](https://sheepbao.github.io/post/golang_code_specification/)
* https://gocn.io/article/1