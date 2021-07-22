###  Go 测试

Go 提倡表格驱动测试

```go

import "testing"

func TestAdd(t *testing.T) {
	tests := []struct{ a, b, c int }{
		{1, 2, 3},
		{2, 3, 5},
		{200, 300, 500},
	}

	for _, tt := range tests {
		if sum := add(tt.a, tt.b); sum != tt.c {
			t.Errorf("error add (a + b) != c  (%d + %d) = %d, got num %d", tt.a, tt.b, sum, tt.c)
		}
	}
}

```
运行命令
```go
~ go test .
```
