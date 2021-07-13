
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
  * new 分配 Allocation with new
    * 它不会初始化内存，只会将内存置零。也就是说，new(T) 会为类型为 T 的新项分配已置零的内存空间， 并返回它的地址，也就是一个类型为 *T 的值。用 Go 的术语来说，它返回一个指针， 该指针指向新分配的，类型为 T 的零值。










