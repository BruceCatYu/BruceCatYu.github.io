---
title: "golang中的context"
tags: ["中文","入门","go","未完结"]
date: 2021-10-25T17:02:29+08:00
draft: false  
---
# golang中的context
位于src/context  
| 包含文件 | 用途 |
| - | - |
| benchmark_test.go | 名字以Benchmark开头的函数,用于性能测试 |
| context.go | context源码,golang1.17中共567行 |
| context_test.go | context.go的测试用例函数 |
| example_test.go | context的使用示例 |
| net_test.go | 与net包配合测试 |
| x_test.go | 用于执行context_test.go的测试用例 |


## 简单示例  
```go
import (
	"context"
	"fmt"
	"time"
)

func main() {
	handleTime := 500 * time.Millisecond // handle函数执行时间
	timeLimit := 1 * time.Second         // context设置超时时间

	ctx, cancel := context.WithTimeout(context.Background(), timeLimit)
	defer cancel()

	go handle(ctx, handleTime)

	<-ctx.Done()
	fmt.Println("main", ctx.Err())
}

func handle(ctx context.Context, handleTime time.Duration) {
	select {
	case <-ctx.Done():
		fmt.Println("handle", ctx.Err())
	case <-time.After(handleTime):
		fmt.Println("process request with", handleTime)
	}
}

```

当函数执行小于超时时间时,输出:  
```zsh
process request with 500ms
main context deadline exceeded
```
当函数执行大于超时时间时,输出有三种情况:  
```zsh
# 结果1
main context deadline exceeded
handle context deadline exceeded

# 结果2
handle context deadline exceeded
main context deadline exceeded

# 结果3
main context deadline exceeded
```

多个goroutine可以接收同一个Context的消息.  


## 总体结构
### 关键属性  
| 名称 | 类型 | 概述 |
| - | - | - |
| Context | 公共接口 | context的核心,为事件定义了四个方法 |
| deadlineExceededError | 私有结构体 | 实现Error接口,用于表示上下文已到达deadline |
| emptyCtx | 类型别名 | int的别名,同时实现了Context接口 |
| canceler | 公共接口 | 定义了取消和完成方法 |
| cancelCtx | 私有结构体 | 实现canceler接口，内嵌了Context |
| timerCtx | 私有结构体 | 组合了cancelCtx |
| CancelFunc | 类型别名 | 无參无返回值的函数别名,用于取消上下文 |
| propagateCancel | 函数 | 用于传递取消信号 |

### 其它属性  
| 名称 | 类型 | 概述 |
| - | - | - |
| Canceled | 公有变量 | error接口的实例,用于表示上下文已取消 |
| DeadlineExceeded | 公有变量 | deadlineExceededError接口的实例 |
| background | 私有变量 | 类型为emptyCtx |
| todo | 私有变量 | 与`background`完全一致,只有名称上的差别 |

### Context接口


------

## 四大With函数  

### WithCancel  
`WithCancel returns a copy of parent with a new Done channel. The returned context's Done channel is closed when the returned cancel function is called or when the parent context's Done channel is closed, whichever happens first.`  
`Canceling this context releases resources associated with it, so code should call cancel as soon as the operations running in this Context complete.`  

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}

```


### WithTimeout  
`WithTimeout returns WithDeadline(parent, time.Now().Add(timeout)).`  
`Canceling this context releases resources associated with it, so code should call cancel as soon as the operations running in this Context complete:`  

```go
func slowOperationWithTimeout(ctx context.Context) (Result, error) {
  ctx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
 	defer cancel()  // releases resources if slowOperation completes before timeout elapses
 	return slowOperation(ctx)
}
```  

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}

```

### WithDeadline  
`WithDeadline returns a copy of the parent context with the deadline adjusted to be no later than d. If the parent's deadline is already earlier than d,WithDeadline(parent, d) is semantically equivalent to parent. The returned context's Done channel is closed when the deadline expires, when the returned cancel function is called, or when the parent context's Done channel is closed, whichever happens first.`  

`Canceling this context releases resources associated with it, so code should call cancel as soon as the operations running in this Context complete.`  

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		// The current deadline is already sooner than the new one.
		return WithCancel(parent)
	}
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
	propagateCancel(parent, c)
	dur := time.Until(d)
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // deadline has already passed
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}

```


### WithValue  
`WithValue returns a copy of parent in which the value associated with key is val.`  
`Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions.`  
`The provided key must be comparable and should not be of type string or any other built-in type to avoid collisions between packages using context. Users of WithValue should define their own types for keys. To avoid allocating when assigning to an interface{}, context keys often have concrete type struct{}.`   
`Alternatively, exported context key variables' static type should be a pointer or interface.`  

```go
func WithValue(parent Context, key, val interface{}) Context {
	if parent == nil {
		panic("cannot create context from nil parent")
	}
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
```


## 和goroutine配合使用 

# References
[context package](https://pkg.go.dev/context?utm_source=gopls)
[学会使用context取消goroutine执行的方法](https://zhuanlan.zhihu.com/p/136664236)
[Golang 并发编程之Context](https://mp.weixin.qq.com/s?__biz=MzUzNTY5MzU2MA==&mid=2247484364&idx=1&sn=31dcd520b7d938f77a04ea79971464c0&chksm=fa80d25bcdf75b4de325fd57fca98198327250e31626ac7339e0141821e153b7579bb4eb94ee&token=298412984&lang=zh_CN#rd)
