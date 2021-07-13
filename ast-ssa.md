
## 抽象语法树

[抽象语法树](https://en.wikipedia.org/wiki/Abstract_syntax_tree) (Abstract Syntax Tree、AST),
是源代码语法的结构的一种抽象表示，它用树状的方式表示编程语言的语法结构。抽象语法树中的每一个节点都表示源代码中的一个元素，
每一颗子树都表示一个语法元素，以表达式 2 * 3 + 7 为例，编译器的语法分析阶段会生成如下图所示的抽象语法树。

![](https://github.com/xtcmoons/go-notes-xtc/blob/main/images/abstract-syntax-tree.png)

## 静态单赋值

[静态单赋值](https://en.wikipedia.org/wiki/Static_single_assignment_form) (Static Single Assignment、SSA) 是中间代码的特性，
如果中间代码具有静态单赋值的特性，那么每个变量就只会被赋值一次。
在实践中，我们通常会用下标实现静态单赋值，这里以下面的代码举个例子：

```text
x := 1
x := 2
y := x
```

经过简单的分析，我们就能够发现上述的代码第一行的赋值语句 x := 1 不会起到任何作用。
下面是具有 SSA 特性的中间代码，我们可以清晰地发现变量 y_1 和 x_1 是没有任何关系的，所以在机器码生成时就可以省去 x := 1 的赋值，通过减少需要执行的指令优化这段代码。

```text
x_1 := 1
x_2 := 2
y_1 := x_2
```

因为 SSA 的主要作用是对代码进行优化，所以它是编译器后端的一部分；当然代码编译领域除了 SSA 还有很多中间代码的优化方法，编译器生成代码的优化也是一个古老并且复杂的领域，这里就不会展开介绍了。

## 词法分析

源代码在计算机 (眼中) 其实是一个由字符组成的，无法被理解的字符串，为了理解这些字符我们需要做的第一件事情就是将字符串分组，
这能够降低理解字符串的成本，简化源代码的分析过程。

```go
make(chan int)
```
可以将上面字符串分成几个部分  <code>make</code>, <code>chan </code>, <code>int </code> 和括号，分解文本的过程就是[词法分析](https://en.wikipedia.org/wiki/Lexical_analysis), 词法分析是将字符串序列转换为标记(token)序列的过程.

Go 语言的词法元素相对来说还是比较简单，使用这种巨大的 switch/case 进行词法解析也比较方便和顺手，早期的 Go 语言虽然使用 lex 这种工具来生成词法解析器，但是最后还是使用 Go 来实现词法分析器，用自己写的词法分析器来解析自己。

## 语法分析
[语法分析](https://en.wikipedia.org/wiki/Parsing) 是根据某种特定的形式文法(Grammar) 对Token 序列构成的输入文本
进行分析并确定其语法结构的过程。从上面的定义来看，词法分析器输出的结果 - Token 序列是语法分析器的输入。



参考

[Go 语言的编译过程](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-compile-intro/)
