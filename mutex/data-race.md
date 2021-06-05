### 如何找到代码中的并发问题

## 我们先看一段代码
```go
1	package main
2
3	import (
4		"fmt"
5		"sync"
6	)
7
8	func main() {
9		var count = 0
10		var wg sync.WaitGroup
11		wg.Add(10)
12		for i := 0; i < 10; i++ {
13			go func() {
14				defer wg.Done()
15				for j := 0; j < 10000; j++ {
16					count++
17				}
18			}()
19		}
20		wg.Wait()
21		fmt.Println(count)
22	}
23
```
在编译(compile) 测试(test) 运行(run) Go 代码的时候，加上`race`参数，就有可能发现并发问题
```
[root@359574908c55 home]# go run -race main.go
==================
WARNING: DATA RACE
Read at 0x0040000b0010 by goroutine 7:
  main.main.func1()
      /home/main.go:16 +0x64

Previous write at 0x0040000b0010 by goroutine 16:
  main.main.func1()
      /home/main.go:16 +0x78

Goroutine 7 (running) created at:
  main.main()
      /home/main.go:13 +0xbc

Goroutine 16 (finished) created at:
  main.main()
      /home/main.go:13 +0xbc
==================
68150
Found 1 data race(s)
exit status 66
```
从这个警告告诉你有并发问题，goroutine 7  对16行有读操作， goroutine 16 对16行有写操作 
开启了race的程序部署在线上，还是比较影响性能的

如何解决 `data race` 的问题 我们可以引入 `sync.Mutex`
```go
     1	package main
     2
     3	import (
     4		"fmt"
     5		"sync"
     6	)
     7
     8	func main() {
     9		var count = 0
    10		var mu sync.Mutex
    11		var wg sync.WaitGroup
    12		wg.Add(10)
    13		for i := 0; i < 10; i++ {
    14			go func() {
    15				defer wg.Done()
    16				for j := 0; j < 10000; j++ {
    17					mu.Lock()
    18					count++
    19					mu.Unlock()
    20				}
    21			}()
    22		}
    23		wg.Wait()
    24		fmt.Println(count)
    25	}
    26
```

