### sync.Mutex 基本用法

1. 第一种
直接写在需要保护的临界区
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

2. 第二种直接嵌入到 struct 中使用
```go
type counter struct {
	sync.Mutex
	Count uint64
}
```

```go
     1	package main
     2
     3	import (
     4		"fmt"
     5		"sync"
     6	)
     7
     8	type counter struct {
     9		sync.Mutex
    10		Count uint64
    11	}
    12
    13	func main() {
    14		var counter = counter{}
    15		var wg sync.WaitGroup
    16		wg.Add(10)
    17		for i := 0; i < 10; i++ {
    18			go func() {
    19				defer wg.Done()
    20				for j := 0; j < 10000; j++ {
    21					counter.Lock()
    22					counter.Count++
    23					counter.Unlock()
    24				}
    25			}()
    26		}
    27		wg.Wait()
    28		fmt.Println(counter.Count)
    29	}
    30
```

如果struct里有多个字段一般把 sync.Mutex 放在要控制的字段上面，用空格把字段隔开
````go
type Counter struct {
    CounterType int
    Name        string
    
    sync.Mutex
    Count uint64
}
````

甚至还可以把需要保护的临界区封装成方法，提供给外部
```go
     1	package main
     2
     3	import (
     4		"fmt"
     5		"sync"
     6	)
     7
     8	type Counter struct {
     9		CounterType int
    10		Name        string
    11
    12		sync.Mutex
    13		count uint64
    14	}
    15
    16	func (c *Counter) Incr() {
    17		c.Lock()
    18		c.count++
    19		c.Unlock()
    20	}
    21
    22	func (c *Counter) Count() uint64 {
    23		c.Lock()
    24		defer c.Unlock()
    25		return c.count
    26	}
    27
    28	func main() {
    29		var counter = Counter{}
    30		var wg sync.WaitGroup
    31		wg.Add(10)
    32		for i := 0; i < 10; i++ {
    33			go func() {
    34				defer wg.Done()
    35				for j := 0; j < 10000; j++ {
    36					counter.Incr()
    37				}
    38			}()
    39		}
    40		wg.Wait()
    41		fmt.Println(counter.Count())
    42	}
    43
```

