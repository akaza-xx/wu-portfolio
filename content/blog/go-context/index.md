---
title: "理解Go里的Context"
date: 2026-07-08
summary: "一文理解Go里的Context"
tags:
  - Go
authors:
  - me
featured: true
---


## 核心概念
- context.Context接口：提供了4个方法
- Deadline: 返回任务的截止时间
- Done:  返回一个channel，用来通知 goroutine 任务是否被取消
- Err：返回取消的原因，比如超时、手动取消
- Value：在请求链接中传递键值对数据

## 常见的创建方式
- 根Context
```go
ctx := context.Background() // 永远不会取消
ctx := context.TODO() // 占位符，表示以后要替换
```

- 可取消Context
```go
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
```

- 带超时Context
```go
ctx, cancel := context.WithTimeout(context.Backgroud(), 2*time.Second)
defer cancel()
```

- 传递值Context
```go
ctx := context.WithValue(context.Background(),"trace_id","abc123")
fmt.Println(ctx.Value("trace_id"))
```

## 使用场景
- http请求：客户端断开时，自动取消后台任务
- 数据库查询：防止长时间阻塞，超时自动取消
- RPC调用：统一传递traceID，用户信息
- 日志追踪：在调用链中传递request-id
- 并发控制：多个goroutine共享一个取消信号

### 演示代码
```go
// 并发控制：多个goroutine共享一个取消信号
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done(): // <-ctx.Done()代表context被取消，这里返回的是一个channel，所以可以用for { select { ... } }语法
			fmt.Printf("Worker %d stopped: %v\n", id, ctx.Err()) // 取消后打印的内容
			return                                               // 取消后要return，否则会一直循环打印取消的内容
		default:
			fmt.Printf("Worker %d working...\n", id)
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second) // 2秒后取消ctx
	defer cancel()                                                          // cancel 是一个函数，由 WithCancel / WithTimeout / WithDeadline 返回,调用它会触发 取消信号，让所有 goroutine 优雅退出。

	go worker(ctx, 1) // 启动goroutine，让函数在后台异步执行，不阻塞主程序。
	go worker(ctx, 2) // 能放到 goroutine 的类型：普通函数、结构体方法、接口方法、匿名函数

	time.Sleep(3 * time.Second)
}

```

```go
// 综合演示代码
package main

import (
	"context"
	"fmt"
	"sync"
	"time"
)

// 模拟数据库查询
func queryDB(ctx context.Context, wg *sync.WaitGroup) {
	defer wg.Done() // 完成后计数减一
	select {
	case <-time.After(1 * time.Second): // 假设查询耗费1秒
		fmt.Println("DB query finished, trace_id:", ctx.Value("trace_id"))
	case <-ctx.Done():
		fmt.Println("DB query canceled:", ctx.Err())
	}
}

func readCache(ctx context.Context, wg *sync.WaitGroup) {
	defer wg.Done() // 完成后计数减一
	select {
	case <-time.After(1 * time.Second):
		fmt.Println("Cache read finished, trace_id:", ctx.Value("trace_id"))
	case <-ctx.Done():
		fmt.Println("Cache read canceled:", ctx.Err())
	}
}

func callRPC(ctx context.Context, wg *sync.WaitGroup) {
	defer wg.Done() // 完成后计数减一
	select {
	case <-time.After(1 * time.Second):
		fmt.Println("callRPC finished, trace_id:", ctx.Value("trace_id"))
	case <-ctx.Done():
		fmt.Println("CallRPC canceled:", ctx.Err())
	}
}

func main() {
	rootCtx := context.Background()

	reqCtx, cancel := context.WithTimeout(rootCtx, 2*time.Second)
	defer cancel()

	reqCtx = context.WithValue(reqCtx, "trace_id", "abc123")

	var wg sync.WaitGroup
	wg.Add(4)

	go queryDB(reqCtx, &wg)
	go readCache(reqCtx, &wg)
	go callRPC(reqCtx, &wg)

	wg.Wait()

	fmt.Println("Request finished, trace_id:", reqCtx.Value("trace_id"))
}

```