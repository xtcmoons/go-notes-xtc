
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
  

* ### 分号 Semicolons
  * Go 的分号，不会出现在源码中。取而代之，词法分析器会使用一条简单的规则来自动插入分号，因此源码中基本不用分号来。
  
### 4 控制结构 Control structures

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
  

  * For
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
















