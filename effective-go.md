
### 1.格式化工具 Formatting

参考 ：[gofmt](https://golang.org/cmd/gofmt/#hdr-Examples)

### 2. 注释 Commentary

* 每个包都应包含一段包注释，即放置在包子句前的一个块注释。
* 注释无需额外格式化
* 若注释总是以名称开头，godoc的命令输出就能通过  `grep` 查找

### 3. 命名 Name

* ### 包名 Package names
  * 包名应当以小写的单词来命名，且不应使用下划线或驼峰记法。
  * 包名就是导入时所需的唯一默认名称，它并不需要在所以源码中保持唯一，
  即便少数发生冲突的情况下，也可为导入的包选择一个别名来局部使用.
  * 另一个约定就是包名应为其源码目录保持一致
  
* ### 获取器 Getters
  * Go 并不对获取器(getter) 和 设置器 (setter) 提供自动支持。
    获取器命名应为 `Owner` (大写，可导出) 而非 `GetOwner`
  * 设置器方法命名 `GetOwner` 是个不错的选择。
  
```go
owner := obj.Owner()
if owner != user {
	obj.SetOwner(user)
}
```

* ###  接口名 Interface names

  * 按照约定，只包含一个方法的接口应当以该方法的名称加上 `-er` 后缀来命名。
  * 若你的类型实现了的方法，于一个众所周知的类型方法拥有相同的含义，那就使用相同的命名。
  如 字符串转换方法命名为 String 而非 ToString
  
* ### 驼峰记法 MixedCaps
  * Go 中约定使用驼峰记法 `MixedCaps` 或 `mixedCaps` 而非下划线的方式对多个单词名称进行命名。
  

### 4. 分号 Semicolons
  * Go 的分号，不会出现在源码中。取而代之，词法分析器会使用一条简单的规则来自动插入分号，因此源码中基本不用分号来。
  

### 5 控制结构 Control structures

* If 
  * In Go a simple if looks like this:
  没有圆括号，而其主体必须始终使用大括号括住
  
```go
if x > 0 {
	return y
}
```
  * `if` `switch` 可接受初始化语句，因此用它们设置局部变量非常常见

```go
if err := file.Chmod(0664); err != nil {
	log.Print(err)
	return err
}
```

* ### 重新声明与再次赋值  Redeclaration and reassignment

  * 简短声明 := 如何使用
  
```go
f, err := os.Open(name)
```
  * 在满足下列条件时，已被声明的变量 v 可出现在 := 声明中
    * 本次声明与已声明的 v 处于同一作用域中
      (若 v 已在外层作用域中声明过，则此次声明会创建一个新的变量)
      
    * 在初始化中与其类型相应的值才能赋予 v, 且在次声明中至少另有一个变量时新声明的
  

* ### For
  * 三种形式的for语句，只有一种需要分号 
  
```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

```go
// 如同 C 的 for 循环
for init; condition; post { }

// 如同 C 的 while 循环
for condition { }

// 如同 C 的 for(;;) 循环
for { }

```

  * Go 没有逗号操作符，而 ++ 和 -- 为语句而非表达式。因此，若你想要在for中使用多个变量，应采用平行赋值的方式

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
	a[i], a[j] = a[j], a[i]
}
```

   * Switch 
        * 表达式无需为常量或者整数
        * switch 并不会自动下溯，但 case 可通过逗号分隔列举相同的处理条件
        * 
    
  * 类型选择  Type switch
    * switch 也可以用于判断接口变量的动态类型，如 类型选择 通过圆括号中的关键字 type 使用类型断言语法。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
	fmt.Printf("unexpected type %T", t)       // %T 输出 t 是什么类型
case bool:
	fmt.Printf("boolean %t\n", t)             // t 是 bool 类型
case int:
	fmt.Printf("integer %d\n", t)             // t 是 int 类型
case *bool:
	fmt.Printf("pointer to boolean %t\n", *t) // t 是 *bool 类型
case *int:
	fmt.Printf("pointer to integer %d\n", *t) // t 是 *int 类型
}
```

### 6. 函数 Functions
  * 多返回值 Multiple return values
```go
func (file *File) Write(b []byte) (n int, err error)
```
  * 命名结果参数 Named result parameters
    * Go 函数的返回值或结果 "形参" 可被命名，并作为常量使用。 
    命名后，一旦函数开始执行，它们就是初始化相应的零值。
    
```go
func ReadFull(r Reader, buf []byte) (n int, err error) {    
	for len(buf) > 0 && err == nil {
		var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}

```
  * Defer
    * 推迟执行函数

```go
// Contents 将文件的内容作为字符串返回。
func Contents(filename string) (string, error) {
	f, err := os.Open(filename)
	if err != nil {
		return "", err
	}
	defer f.Close()  // f.Close 会在我们结束后运行。

	var result []byte
	buf := make([]byte, 100)
	for {
		n, err := f.Read(buf[0:])
		result = append(result, buf[0:n]...) // append 将在后面讨论。
		if err != nil {
			if err == io.EOF {
				break
			}
			return "", err  // 我们在这里返回后，f 就会被关闭。
		}
	}
	return string(result), nil // 我们在这里返回后，f 就会被关闭。
}
```
  * 被推迟函数的实参（如果该函数为方法则还包括接收者）在推迟执行时就会被求值， 而不是在调用执行时才求值。

```go
for i := 0; i < 5; i++ {
	defer fmt.Printf("%d ", i)
}
```

  * 被推迟的函数按照后进先出（LIFO）的顺序执行，因此以上代码在函数返回时会打印 4 3 2 1 0。

```go

```

### 7. 数据 Data
  * ### new 分配 Allocation with new
    * 它不会初始化内存，只会将内存置零。也就是说，new(T) 会为类型为 T 的新项分配已置零的内存空间， 并返回它的地址，也就是一个类型为 *T 的值。用 Go 的术语来说，它返回一个指针， 该指针指向新分配的，类型为 T 的零值。
    * “零值属性” 是传递性的。

```go
type SyncedBuffer struct {
	lock    sync.Mutex
	buffer  bytes.Buffer
}
```

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

  * ### 构造函数与复合字面量 Constructors and composite literals
    * 代码过于冗长。我们可通过复合字面量来简化它
    
```go
func NewFile(fd int, name string) *File {
	if fd < 0 {
		return nil
	}
	f := File{fd, name, nil, 0}
	return &f
}
```

  * 请注意，返回一个局部变量的地址完全没有问题，这点与 C 不同。复合字面的字段必须按顺序全部列出。
  * 表达式 `new(File)` 和 `&File{}` 是等价的。
  
  * ### 使用 `make` 分配  Allocation with make
    * 它只用于创建切片、映射和信道，并返回类型为 T（而非 *T）的一个已初始化 （而非置零）的值。
    * 对于切片、映射和信道，make 用于初始化其内部的数据结构并准备好将要使用的值。

```go
var p *[]int = new([]int)       // 分配切片结构；*p == nil；基本没用
var v  []int = make([]int, 100) // 切片 v 现在引用了一个具有 100 个 int 元素的新数组

// 没必要的复杂：
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// 习惯用法：
v := make([]int, 100)

```

  * ### 数组 Arrays
  * 主要用作切片的构件。
  * 以下为数组在 Go 和 C 中的主要区别。在 Go 中，
    * 数组是值。将一个数组赋予另一个数组会复制其所有元素。
    * 特别地，若将某个数组传入某个函数，它将接收到该数组的一份副本而非指针。
    * 数组的大小是其类型的一部分。类型 [10]int 和 [20]int 是不同的。
    
  * ### 切片 Slices
   * Go 中的大部分数组编程都是通过切片来完成的。
   * 切片保存了对底层数组的引用，若你将某个切片赋予另一个切片，它们会引用同一个数组。
   * 若某个函数将一个切片作为参数传入，则它对该切片元素的修改对调用者而言同样可见， 这可以理解为传递了底层数组的指针。


  * ### 映射 Maps
    * 映射，其键可以是是任何相等性操作符支持的类型，如 整数，浮点数，复数，字符串，指针，接口(只要动态类型支持相等性判断)
    切片不能用作映射键，因为它们的相等性还没定义。
      
   * 与切片一样，映射也是引用类型。
   * 若试图通过映射中不存在的键来取值，就会返回与该映射中项的类型对应的零值。
   * 有时你需要区分莫项是 `不存在`  或 `零值`,  你可以使用多重赋值的形式来分辨这种情况, 可以称为 "逗号 ok"
```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

### 7. 初始化 Initialization

* ### 常量
  * Go 中的常量就是不变量. 它们在编译时创建，即便它们可能是函数中的局部变量。
  * 常量只能是数字、字符（符文）、字符串或布尔值。
  * 由于编译时的限制， 定义它们的表达式必须也是可被编译器求值的常量表达式。  
    
* ### 变量  Variables
  * 变量的初始化与常量类似，但其初始值也可以是在运行时才被计算的一般表达式。

* ### init 函数  The init function
  * 每个源文件都可以通过定义自己的无参数 init 函数来设置一些必要的状态。
  * 其实每个文件都可以拥有多个 init 函数。   
  * 只有该包中的所以变量声明都通过初始化求值后 init 函数才会调用
  * 而那些 init 函数只有在所有已导入包都被初始化后才会被求值 
    
### 8. 方法 Methods
* 指针 vs .值 Pointers vs .Values
  * 可以为任何类型定义方法
    
### 9. 接口和其他类型  Interfaces and other types
* ### 接口
  * 每种类型都能实现多个接口。
    
```go
type Sequence []int

// Methods required by sort.Interface.
// sort.Interface 所需的方法。
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Method for printing - sorts the elements before printing.
// 用于打印的方法 - 在打印前对元素进行排序。
func (s Sequence) String() string {
    sort.Sort(s)
    str := "["
    for i, elem := range s {
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}


```
  
* ### 类型转换  Conversions
```go

type Sequence []int

// // 用于打印的方法 - 在打印前对元素进行排序。
func (s Sequence) String() string {
	sort.IntSlice(s).Sort()
	return fmt.Sprint([]int(s))
}

```

* ### 接口转换与类型断言  Interface conversions and type assertions
  * 类型选择 是类型转换的一种形式
    
```go
type Stringer interface {
	String() string
}

var value interface{} // 调用者提供的值。
switch str := value.(type) {
case string:
	return str
case Stringer:
	return str.String()
}

```
  * 类型断言接受一个接口值， 并从中提取指定的明确类型的值。其语法借鉴自类型选择开头的子句，但它需要一个明确的类型， 而非 type 关键字：
```go
value.(typeName)
```

```go
str := value.(string)

```
  * 若类型断言失败，str 将继续存在且为字符串类型，但它将拥有零值，即空字符串。

```go
if str, ok := value.(string); ok {
	return str
} else if str, ok := value.(Stringer); ok {
	return str.String()
}
```

* ### 接口和方法 Interfaces and methods
  * 几乎任何类型都能添加方法，因此几乎任何类型都能满足一个接口。
    
### 10. 空白标识符 The blank identifier
  * 用空白标识符来代替该变量可避免创建无用的变量，并能清楚地表明该值将被丢弃。
```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```
 * 未使用的导入和变量
```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done. // 用于调试，结束时删除。
var _ io.Reader    // For debugging; delete when done. // 用于调试，结束时删除。

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}

```

  * 未副作用而导入 Import for side effect
    * 有时导入某个包只是为了其副作用， 而没有任何明确的使用。只需将该包重命名为空白标识符：
    
```go
import _ "net/http/pprof"

```
  * 若只需要判断某个类型是否是实现了某个接口，而不需要实际使用接口本身 （可能是错误检查部分），就使用空白标识符来忽略类型断言的值：
```go
if _, ok := val.(json.Marshaler); ok {
	fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}

```

### 11. 内嵌 Embedding 
```go
type Reader interface {
	Read(p []byte) (n int, err error)
}

type Writer interface {
	Write(p []byte) (n int, err error)
}

```

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
	Reader
	Writer
}

```

 * 同样的基本想法可以应用在结构体中
```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
	*Reader  // *bufio.Reader
	*Writer  // *bufio.Writer
}
```
  * 当内嵌一个类型时，该类型的方法会成为外部类型的方法， 但当它们被调用时，该方法的接收者是内部类型，而非外部的。
```go
type Job struct {
	Command string
	*log.Logger
}
```

```go
job.Log("starting now...")

```

### 12. 并发 Concurrency
  * 通过通信共享内存 Share by communicating
```text
Do not communicate by sharing memory. instead, share monery by communicating.

不要通过共享内存来通信，而应通过通信来共享内存。
```

  * Goroutines
    * 在 Go 中，函数字面都是闭包：其实现在保证了函数内引用变量的生命周期与函数的活动时间相同。

  * 信道 Channels
```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files

```
  * 用法
```go
c := make(chan int)  // 分配一个信道
// 在 goroutine 中启动排序。当它完成后，在信道上发送信号。
go func() {
	list.Sort()
	c <- 1  // 发送信号，什么值无所谓。
}()
doSomethingForAWhile()
<-c   // 等待排序结束，丢弃发来的值。

```
 * 接收者在收到数据前会一直阻塞。若信道是不带缓冲的，那么在接收者收到值前， 发送者会一直阻塞；若信道是带缓冲的，则发送者直到值被复制到缓冲区才开始阻塞； 若缓冲区已满，发送者会一直等待直到某个接收者取出一个值为止。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
	sem <- 1 // 等待活动队列清空。
	process(r)  // 可能需要很长时间。
	<-sem    // 完成；使下一个请求可以运行。
}

func Serve(queue chan *Request) {
	for {
		req := <-queue
		go handle(req)  // 无需等待 handle 结束。
	}
}

```

  * 信道中的信道 Channels of channels
    * Go 最重要的特性就是信道是一等值，它可以被分配并像其它值到处传递。 这种特性通常被用来实现安全、并行的多路分解。
    
  * ### 并行化 Parallelization

### 13. 错误 Error
  * 按照约定，错误的类型通常为 error，这是一个内建的简单接口。
```go
type error interface {
	Error() string
}

```
