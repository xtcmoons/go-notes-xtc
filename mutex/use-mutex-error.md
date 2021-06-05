### 常见使用Mutex: 错误场景

1. Lock/Unlock 不是成对出现

2. Copy 已使用的Mutex
```go
     1  package main
     2  
     3  import (
     4          "fmt"
     5          "sync"
     6  )
     7  
     8  type Counter struct {
     9          sync.Mutex
    10          Count int
    11  }
    12  
    13  func main() {
    14          var counter Counter
    15          counter.Lock()
    16          defer counter.Unlock()
    17          counter.Count++
    18          foo(&counter)
    19  }
    20  
    21  func foo(c *Counter) {
    22          c.Lock()
    23          defer c.Unlock()
    24          fmt.Println("in foo")
    25  }

```

3. 重入
   当一个线程获取锁时，如果没有其他线程拥有这个锁，那么这个线程就成功获取到这个锁。 
   之后，如果其他线程再请求这个锁，就会出入阻塞等待的状态。但是，如果拥有这把
   锁的线程再请求这个把锁的话，不会阻塞，而是成功返回，所以叫可重入锁（有时候
   也叫做递归锁)。
   
## 划重点 Mutex 不是可重入锁
   因为Mutex的实现中没有记录那个goroutine拥有这把锁。理论上，任何goroutine都可以
   随意地 Unlock 这把锁，所以没办法计算重入条件。
   
```go
     1  package main
     2  
     3  import (
     4          "fmt"
     5          "sync"
     6  )
     7  
     8  func foo(l sync.Locker) {
     9          fmt.Println("in foo")
    10          l.Lock()
    11          bar(l)
    12          l.Unlock()
    13  }
    14  
    15  func bar(l sync.Locker) {
    16          l.Lock()
    17          fmt.Println("in bar")
    18          l.Unlock()
    19  }
    20  
    21  func main() {
    22          l := &sync.Mutex{}
    23          foo(l)
    24  }

```

4. 死锁
产生死锁的必要条件
* 互斥 --- 至少一个资源是被排他性独享的，其他线程必须处于等待状态，直到资源被释放。
* 持有和等待 --- goroutine 持有一个资源，并且还在请求其他 goroutine 持有的资源，也就是咱们常说的
  "吃着碗里，看这锅里"的意思
* 不可剥夺 --- 资源只能由持有它的 goroutine 来释放
* 环路等待 --- 一般来说，存在一组进程，P={P1,P2,..., PN}, P1等待P2持有的资源，P2等待P3持有的资源，
  依次类推，最后是PN等待P1持有的资源，这就形成了一个环路等待的死结.




























