---
title: Go 简明教程
date: 2020-03-28 08:24:18
tags: [golang]
categories:
top_img:
---

<a name="48lfj"></a>
### Intro
> Go（又称 Golang）是 Google 开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言。 —— Go - wikipedia.org

<!--more-->

<a name="ZwhID"></a>
### Hello World
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello World!")
}
```


```bash
# ronething @ ashings-macbook-pro in ~/Documents/golang-sample [12:01:24] 
$ ls    
go.mod  main.go

# ronething @ ashings-macbook-pro in ~/Documents/golang-sample [12:01:25] 
$ go build main.go                                                           

# ronething @ ashings-macbook-pro in ~/Documents/golang-sample [12:01:29] 
$ ls 
go.mod  main    main.go

# ronething @ ashings-macbook-pro in ~/Documents/golang-sample [12:01:30] 
$ ./main               
Hello World!
```


<a name="6c72V"></a>
### 变量与内置数据类型

- 变量(Variable)

Go 语言是静态类型的，变量声明时必须明确变量的类型。Go 语言与其他语言显著不同的一个地方在于，Go 语言的类型在变量后面。比如 java 中，声明一个整体一般写成 `int a = 1`，在 Go 语言中，需要这么写

```go
var a int // 如果没有赋值，默认为0
var a int = 1 // 声明时赋值
var a = 1 // 声明时赋值
```

`var a = 1`，因为 1 是 int 类型的，所以赋值时，a 自动被确定为 int 类型，所以类型名可以省略不写，这种方式还有一种更简单的表达

```go
a := 1
msg := "hello world"
```


- 简单类型

空值：nil整型类型： int(取决于操作系统), int8, int16, int32, int64, uint8, uint16, …浮点数类型：float32, float64字节类型：byte (等价于uint8)字符串类型：string布尔值类型：boolean，(true 或 false)

```go
var a int8 = 10
var c1 byte = 'a'
var b float32 = 12.2
var msg = "Hello World"
ok := false
```


- 字符串

在 Go 语言中，字符串使用 UTF8 编码，UTF8 的好处在于，如果基本是英文，每个字符占 1 byte，和 ASCII 编码是一样的，非常节省空间，如果是中文，一般占3字节。包含中文的字符串的处理方式与纯 ASCII 码构成的字符串有点区别。

```go
package main

import (
	"fmt"
	"reflect"
)
func main() {
    str1 := "Golang"
    str2 := "Go语言"
    fmt.Println(reflect.TypeOf(str2[2]).Kind()) // uint8
    fmt.Println(str1[2], string(str1[2]))       // 108 l
    fmt.Printf("%d %c\n", str2[2], str2[2])     // 232 è
    fmt.Println("len(str2)：", len(str2))       // len(str2)： 8
}
```


- reflect.TypeOf().Kind() 可以知道某个变量的类型，我们可以看到，字符串是以 byte 数组形式保存的，类型是 uint8，占1个 byte，打印时需要用 string 进行类型转换，否则打印的是编码值。
- 因为字符串是以 byte 数组的形式存储的，所以，`str2[2]` 的值并不等于`语`。str2 的长度 `len(str2)` 也不是 4，而是 8（ Go 占 2 byte，语言占 6 byte）



> 将字符串转为 rune 数组

```go
str2 := "Go语言"
runeArr := []rune(str2)
fmt.Println(reflect.TypeOf(runeArr[2]).Kind()) // int32
fmt.Println(runeArr[2], string(runeArr[2]))    // 35821 语
fmt.Println("len(runeArr)：", len(runeArr))    // len(runeArr)： 4
```

转换成 `[]rune` 类型后，字符串中的每个字符，无论占多少个字节都用 int32 来表示，因而可以正确处理中文。

- 数组(array)与切片(slice)


声明数组

```go
var arr [5]int     // 一维
var arr2 [5][5]int // 二维
```

声明时初始化

```go
var arr = [5]int{1, 2, 3, 4, 5}
arr := [5]int{1, 2, 3, 4, 5}
```

使用 `[]` 索引/修改数组

```go
arr := [5]int{1, 2, 3, 4, 5}
for i := 0; i < len(arr); i++ {
	arr[i] += 100
}
fmt.Println(arr)  // [101 102 103 104 105]
```

数组的长度不能改变，如果想拼接 2 个数组，或是获取子数组，需要使用切片。切片是数组的抽象。 切片使用数组作为底层结构。切片包含三个组件：容量，长度和指向底层数组的指针,切片可以随时进行扩展
声明切片：
```go
slice1 := make([]float32, 0) // 长度为0的切片
slice2 := make([]float32, 3, 5) // [0 0 0] 长度为 3，容量为 5 的切片 capacity
fmt.Println(len(slice2), cap(slice2)) // 3 5
```

使用切片：

```go
// 添加元素，切片容量可以根据需要自动扩展
slice2 = append(slice2, 1, 2, 3, 4) // [0, 0, 0, 1, 2, 3, 4]
fmt.Println(len(slice2), cap(slice2)) // 7 12
// 子切片 [start, end)
sub1 := slice2[3:] // [1 2 3 4]
sub2 := slice2[:3] // [0 0 0]
sub3 := slice2[1:4] // [0 0 1]
// 合并切片
combined := append(sub1, sub2...) // [1, 2, 3, 4, 0, 0, 0]
```


- 声明切片时可以为切片设置容量大小，为切片预分配空间。在实际使用的过程中，如果容量不够，切片容量会自动扩展。
- `sub2...` 是切片解构的写法，将切片解构为 N 个独立的元素


字典(键值对, map)
> map 类似于 java 的 HashMap，Python的字典(dict)，是一种存储键值对(Key-Value)的数据解构。

```go
// 仅声明
m1 := make(map[string]int)
// 声明时初始化
m2 := map[string]string{
	"Sam": "Male",
	"Alice": "Female",
}
// 赋值/修改
m1["Tom"] = 18
```

指针(Pointer)

```go
str := "Golang"
var p *string = &str // p 是指向 str 的指针
*p = "Hello"
fmt.Println(str) // Hello 修改了 p，str 的值也发生了改变
```


```go
func add(num int) {
	num += 1
}

func realAdd(num *int) {
	*num += 1
}

func main() {
	num := 100
	add(num)
	fmt.Println(num)  // 100，num 没有变化

	realAdd(&num)
	fmt.Println(num)  // 101，指针传递，num 被修改
}
```


<a name="CVS40"></a>
### 流程控制(if, for, switch)


```go
age := 18
if age < 18 {
	fmt.Printf("Kid")
} else {
	fmt.Printf("Adult")
}

// 可以简写为：
if age := 18; age < 18 {
	fmt.Printf("Kid")
} else {
	fmt.Printf("Adult")
}
```


```go
type Gender int8
const (
	MALE   Gender = 1
	FEMALE Gender = 2
)

gender := MALE

switch gender {
case FEMALE:
	fmt.Println("female")
case MALE:
	fmt.Println("male")
default:
	fmt.Println("unknown")
}
// male
```

和其他语言不同的地方在于，Go 语言的 switch 不需要 break，匹配到某个 case，执行完该 case 定义的行为后，默认不会继续往下执行。如果需要继续往下执行，需要使用 fallthrough，例如：
```go
switch gender {
case FEMALE:
	fmt.Println("female")
	fallthrough
case MALE:
	fmt.Println("male")
	fallthrough
default:
	fmt.Println("unknown")
}
// 输出结果
// male
// unknown
```


```go
sum := 0
for i := 0; i < 10; i++ {
	if sum > 50 {
		break
	}
	sum += i
}
```


```go
nums := []int{10, 20, 30, 40}
for i, num := range nums {
	fmt.Println(i, num)
}
// 0 20
// 1 30
// 2 40
m2 := map[string]string{
	"Sam":   "Male",
	"Alice": "Female",
}

for key, value := range m2 {
	fmt.Println(key, value)
}
// Sam Male
// Alice Female
```


<a name="LsNcm"></a>
### 函数(functions)

一个典型的函数定义如下，使用关键字 `func`，参数可以有多个，返回值也支持有多个。特别地，`package main` 中的`func main()` 约定为可执行程序的入口。
参数与返回值

```go
func funcName(param1 Type1, param2 Type2, ...) (return1 Type3, ...) {
    // body
}
```

也可以给返回值命名，简化 return，例如 add 函数可以改写为
```go
func add(num1 int, num2 int) (ans int) {
	ans = num1 + num2
	return
}
```

错误处理(error handling)
如果函数实现过程中，如果出现不能处理的错误，可以返回给调用者处理。


```go
import (
	"fmt"
	"os"
)

func main() {
	_, err := os.Open("filename.txt")
	if err != nil {
		fmt.Println(err)
	}
}

// open filename.txt: no such file or directory
```

通过 errors.New 返回自定义的错误

```go
import (
	"errors"
	"fmt"
)

func hello(name string) error {
	if len(name) == 0 {
		return errors.New("error: name is null")
	}
	fmt.Println("Hello,", name)
	return nil
}

func main() {
	if err := hello(""); err != nil {
		fmt.Println(err)
	}
}
// error: name is null
```

error 往往是能预知的错误，但是也可能出现一些不可预知的错误，例如数组越界，这种错误可能会导致程序非正常退出，在 Go 语言中称之为 `panic` 。

```go
func get(index int) int {
	arr := [3]int{2, 3, 4}
	return arr[index]
}

func main() {
	fmt.Println(get(5))
	fmt.Println("finished")
}
```


```bash
$ go run .
panic: runtime error: index out of range [5] with length 3
goroutine 1 [running]:
exit status 2
```

在 Python、Java 等语言中有 `try...catch` 机制，在 `try` 中捕获各种类型的异常，在 `catch` 中定义异常处理的行为。Go 语言也提供了类似的机制 `defer` 和 `recover`。

```go
func get(index int) (ret int) {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("Some error happened!", r)
			ret = -1
		}
	}()
	arr := [3]int{2, 3, 4}
	return arr[index]
}

func main() {
	fmt.Println(get(5))
	fmt.Println("finished")
}
```

```bash
$ go run .
Some error happened! runtime error: index out of range [5] with length 3
-1
finished
```


- 在 get 函数中，使用 defer 定义了异常处理的函数，在协程退出前，会执行完 defer 挂载的任务。因此如果触发了 panic，控制权就交给了 defer。
- 在 defer 的处理逻辑中，使用 recover，使程序恢复正常，并且将返回值设置为 -1，在这里也可以不处理返回值，如果不处理返回值，返回值将被置为默认值 0。

<a name="kpSWP"></a>
### 结构体、方法和接口

- 结构体(struct) 和方法(methods)

结构体类似于其他语言中的 class，可以在结构体中定义多个字段，为结构体实现方法，实例化等。

```go
type Student struct {
	name string
	age  int
}

func (stu *Student) hello(person string) string { // 结构体方法
	return fmt.Sprintf("hello %s, I am %s", person, stu.name)
}

func main() {
	stu := &Student{
		name: "Tom",
	}
	msg := stu.hello("Jack")
	fmt.Println(msg) // hello Jack, I am Tom
}
```


- 使用 `Student{field: value, ...}`的形式创建 Student 的实例，字段不需要每个都赋值，没有显性赋值的变量将被赋予默认值，例如 age 将被赋予默认值 0。
- **实现方法与实现函数的区别**在于，`func` 和函数名`hello` 之间，加上该方法对应的实例名 `stu` 及其类型 `*Student`，可以通过实例名访问该实例的字段`name`和其他方法了。
- 调用方法通过 `实例名.方法名(参数)` 的方式。



```go
// 也可以使用 new 进行实例化
func main() {
	stu2 := new(Student)
	fmt.Println(stu2.hello("Alice")) // hello Alice, I am  , name 被赋予默认值""
}
```


- 接口(interfaces)


一般而言，接口定义了一组方法的集合，接口不能被实例化，一个类型可以实现多个接口。

```go
// Student 实现 Person 以及 Teen interface

type Person interface {
	getName() string
}

type Teen interface {
	getAge() int
}

type Student struct {
	name string
	age  int
}

func (stu *Student) getName() string {
	return stu.name
}

func (stu *Student) getAge() int {
	return stu.age
}

func main() {
	s := Student{
		name: "Tom",
		age:  18,
	}

	var p Person = &s
	var t Teen = &s

	fmt.Println(p.getName()) // Tom
	fmt.Println(t.getAge())  // 18
}
```

Go 语言中，并不需要显式地声明实现了哪一个接口，只需要直接实现该接口对应的方法即可。
Q: 我们如何确保某个类型实现了某个接口的所有方法呢？一般可以使用下面的方法进行检测，如果实现不完整，编译期将会报错。

```go
// 将空值 nil 转换为 *Student 类型，再转换为 Person 接口，如果转换失败，说明 Student 并没有实现 Person 接口的所有方法。
var _ Person = (*Student)(nil)
```

实例可以强制类型转换为接口，接口也可以强制类型转换为实例。
```go
func main() {
	var p Person = &Student{
		name: "Tom",
		age:  18,
	}

	stu := p.(*Student) // 接口转为实例
	fmt.Println(stu.getAge())
}
```


- 空接口


如果定义了一个没有任何方法的空接口，那么这个接口可以表示任意类型。例如

```go
func main() {
	m := make(map[string]interface{})
	m["name"] = "Tom"
	m["age"] = 18
	m["scores"] = [3]int{98, 99, 85}
	fmt.Println(m) // map[age:18 name:Tom scores:[98 99 85]]
}
```


<a name="Deb1Y"></a>
### 并发编程(goroutine)

- sync

Go 语言提供了 sync 和 channel 两种方式支持协程 (goroutine) 的并发。例如我们希望并发下载 N 个资源，多个并发协程之间**不需要**通信，那么就可以使用 sync.WaitGroup，等待所有并发协程执行结束。

```go
import (
	"fmt"
	"sync"
	"time"
)

var wg sync.WaitGroup

func download(url string) {
	fmt.Println("start to download", url)
	time.Sleep(time.Second) // 模拟耗时操作
	wg.Done()
}

func main() {
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go download("a.com/" + string(i+'0'))
	}
	wg.Wait()
	fmt.Println("Done!")
}
```


- wg.Add(1)：为 wg 添加一个计数，wg.Done()，减去一个计数。
- go download()：启动新的协程并发执行 download 函数。
- wg.Wait()：等待所有的协程执行结束。



```bash
# ronething @ ashings-macbook-pro in ~/Documents/golang-sample [21:31:39] 
$ time go run wait.go      
start to download a.com/2
start to download a.com/1
start to download a.com/0
Done!
go run wait.go  0.29s user 0.22s system 36% cpu 1.387 total
```


- channel: 如果协程之间**需要**通信，可以考虑需要 channel



```go
var ch = make(chan string, 10) // 创建大小为 10 的缓冲信道

func download(url string) {
	fmt.Println("start to download", url)
	time.Sleep(time.Second)
	ch <- url // 将 url 发送给信道
}

func main() {
	for i := 0; i < 3; i++ {
		go download("a.com/" + string(i+'0'))
	}
	for i := 0; i < 3; i++ {
		msg := <-ch // 等待信道返回消息。
		fmt.Println("finish", msg)
	}
	fmt.Println("Done!")
}
```


```bash
# ronething @ ashings-macbook-pro in ~/Documents/golang-sample [21:33:55] 
$ time go run channel.go 
start to download a.com/2
start to download a.com/0
start to download a.com/1
finish a.com/2
finish a.com/0
finish a.com/1
Done!
go run channel.go  0.28s user 0.23s system 36% cpu 1.386 total
```

使用 channel 信道，可以在协程之间传递消息。阻塞等待并发协程返回消息。
<a name="h1yq7"></a>
### 单元测试(unit test)
假设我们希望测试 package main 下 `calc.go` 中的函数，要只需要新建 `calc_test.go` 文件，在`calc_test.go`中新建测试用例即可。

```go
// calc.go
package main

func add(num1 int, num2 int) int {
	return num1 + num2
}

// calc_test.go
package main

import "testing"

func TestAdd(t *testing.T) {
	if ans := add(1, 2); ans != 3 {
		t.Error("add(1, 2) should be equal to 3")
	}
}
```

运行 `go test`，将自动运行当前 package 下的所有测试用例，如果需要查看详细的信息，可以添加`-v`参数。

```bash
$ go test -v                  
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
ok  	github.com/ronething/go-sample/gotest	0.005s
```


<a name="AlYhD"></a>
### 包(package)和模块(module)

- package

一般来说，一个文件夹可以作为 package，同一个 package 内部变量、类型、方法等定义可以相互看到。比如我们新建一个文件 calc.go， main.go 平级，分别定义 add 和 main 方法。
```go
// calc.go
package main

func add(num1 int, num2 int) int {
	return num1 + num2
}

// main.go
package main

import "fmt"

func main() {
	fmt.Println(add(3, 5)) // 8
}
```
运行 go run main.go，会报错，add 未定义：`./main.go:6:14: undefined: add` 因为 go run main.go 仅编译 main.go 一个文件，所以命令需要换成
```bash
$ go run main.go calc.go
8
### or

$ go run .
8
```

Go 语言也有 Public 和 Private 的概念，粒度是包。如果类型/接口/方法/函数/字段的首字母大写，则是 Public 的，对其他 package 可见，如果首字母小写，则是 Private 的，对其他 package 不可见。

- module

[Go Modules](https://github.com/golang/go/wiki/Modules) 是 Go 1.11 版本之后引入的，Go 1.11 之前使用 $GOPATH 机制。Go Modules 可以算作是较为完善的包管理工具。同时支持代理，国内也能享受高速的第三方包镜像服务。接下来简单介绍 `go mod` 的使用。Go Modules 在 1.13 版本仍是可选使用的，环境变量 GO111MODULE 的值默认为 AUTO，强制使用 Go Modules 进行依赖管理，可以将 GO111MODULE 设置为 ON。

```bash
## 初始化项目
$ go mod init example
go: creating new go.mod: module example
```


```go
package main

import (
	"fmt"

	"rsc.io/quote"
)

func main() {
	fmt.Println(quote.Hello())  // Ahoy, world!
}
```


```
### go.mod 文件
module example

go 1.13

require rsc.io/quote v3.1.0+incompatible
```


```bash
# go mod 一些基础命令
go mod init [module]
go mod tidy
go mod download
```


<a name="WSppT"></a>
### 参考资料
[https://geektutu.com/post/quick-golang.html](https://geektutu.com/post/quick-golang.html)
