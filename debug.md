## 中间代码

使用下面命令将Go的源码编译成汇编语言
```shell
$ go build -gcflags -S main.go
```

如果想要了解Go语言更详细的编译过程，通过下面命令获取汇编指令
```shell
$ GOSSAFUNC=main go build main.go

# runtime
dumped SSA to ~/workspace/ssa.html
# command-line-arguments
dumped SSA to ./ssa.html

```
![]("./images/ssa.png")

上述命令会在当前文件夹下生成一个ssa.html文件，打开这个文件就能看到汇编优化的每一个步骤