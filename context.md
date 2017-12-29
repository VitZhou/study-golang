# Package context

```go
import "context"
```

## 概述


Package context定义了Context类型,该Context类型跨越api边界和进程之间传递deadline,cancelation信号和其他请求范围的值。

对服务器的传入请求应该创建一个Context，而对服务器的传承怒调用应该接受一个Context.它们之间的函数调用链必须传播上下文,可以使用WithCancel,WithDeadline,WithTimeout或WithValue创建的派生Context来替换它。当Context被取消时,它从派生的所有上下文也被取消.

WithCancel，WithDeadline和WithTimeout函数采用Context(父级)并返回派生的Context(子级)和CancelFunc。调用CancelFunc将取消子项及其子项，删除父项对子项的引用,并停止任何关联的定时器.未能调用CancelFunc的会泄漏子对象及其子对象，直到父对象被取消或定时器触发。go vet工具检查在所有控制流路径上是否使用CancelFuncs。

使用Context的程序应遵循这些规则，以保持包之间的接口一致，并使静态分析工具能够检查Context传播：

不要将Context存储在struct类型中; 而是将一个Context明确地传递给每个需要它的函数。 Context应该是第一个参数，通常命名为ctx：

```go
func DoSomething(ctx context.Context, arg Arg) error {
	// ... use ctx ...
}
```

即使函数允许，也不要传递一个nil Context。 如果您不确定要使用哪个Context，请传递context.TODO。

使用Context值仅适用于传输进程和API的请求范围数据，而不是将可选参数传递给函数。

相同的Context可以传递给在不同的goroutine中运行的函数。 Context对于多个goroutine同时使用是安全的。

## 变量

Canceled是Context取消时由Context.Err返回的错误。

```go
var Canceled = errors.New("context canceled")
```

DeadlineExceeded是Context.Err在Context 超过deadline时返回的错误。

```go
var DeadlineExceeded error = deadlineExceededError{}
```

## type CancelFunc

CancelFunc告诉操作放弃其工作。 CancelFunc不会等待工作停止。 在第一次调用之后，对CancelFunc的后续调用什么也不做。

```go
type CancelFunc func()
```

## type Context

Context包含deadline，cancelation信号以及跨越API边界的其他值。

Context的方法可以被多个goroutine同时调用。

```go
type Context interface {
        //  Deadline返回代表这个Context的工作应该被取消的时间。 如果没有设置deadline，Deadline返回ok == false。 对			// Deadline的连续调用返回相同的结果。
        Deadline() (deadline time.Time, ok bool)

        // Done返回一个channel，当这个Context完成的工作应该被取消时它会关闭。如果这个Context永远不能被取
        // 消，那么Done可能返回nil。连续调用完成返回相同的值。
        //
        // 取消调用时，WithCancel安排Done关闭; WithDeadline安排在deadline到期时完成关闭; WithTimeout
        // 在超时时间过后安排Done关闭。
        //
        // 提供用于select语句的Done:
        //
        //  //  Stream使用DoSomething生成值并将其发送出去
        // //直到DoSomething返回一个错误或者ctx.Done被关闭。
        //  func Stream(ctx context.Context, out chan<- Value) error {
        //  	for {
        //  		v, err := DoSomething(ctx)
        //  		if err != nil {
        //  			return err
        //  		}
        //  		select {
        //  		case <-ctx.Done():
        //  			return ctx.Err()
        //  		case out <- v:
        //  		}
        //  	}
        //  }
        //
        // 有关如何使用“Done”通道进行取消的更多示例，请参阅https://blog.golang.org/pipelines。
        Done() <-chan struct{}

        // 如果Done还没有关闭，则Err返回nil。
        // 如果完成关闭，则Err返回一个非零错误，解释原因：
        // 如果Context被取消，则取消;如果Context的Deadline已过，则取消DeadlineExceeded。Err返回非零错
        // 误后，对Err的连续调用将返回相同的错误。
        Err() error

        // Value返回与此Context关联的值，如果没有value与Key关联，则返回nil。 用相同的key连续调用Value返
        // 回相同的结果。
        //
        // 仅将Context Value用于传输进程和API边界的请求范围数据，而不是将可选参数传递给函数。
        //
        // key标识Context中的特定value。 希望在Context中存储value的函数通常在全局变量中分配一个key，然
        // 后使用该key作为context.WithValue和Context.Value的参数。 一个key可以是任何支持equality的类
        // 型;包应该将key定义为未导出的类型以避免碰撞。
        //
        // 定义Context key的package应该为使用该key存储的value提供类型安全的访问器：
        //
        // 	// package user定义存储在Context中的user类型
        // 	package user
        //
        // 	import "context"
        //
        // 	// User是存储在Context中value的类型
        // 	type User struct {...}
        //
        // 	//key是此包中定义的键的未导出类型。这可以防止与其他包中定义的键的冲突。
        // 	type key int
        //
        // 	// userKey是user的key。Context的user value。 它是未经导出的; 客户端使用user.NewContext和user.FromContext而不是直接使用这个key。
        // 	var userKey key = 0
        //
        // 	// NewContext返回一个新的有值u的Context.
        // 	func NewContext(ctx context.Context, u *User) context.Context {
        // 		return context.WithValue(ctx, userKey, u)
        // 	}
        //
        // 	// FromContext返回存储在ctx中的用户值，如果有的话。
        // 	func FromContext(ctx context.Context) (*User, bool) {
        // 		u, ok := ctx.Value(userKey).(*User)
        // 		return u, ok
        // 	}
        Value(key interface{}) interface{}
}
```

## func [Background](https://golang.org/src/context/context.go?s=7306:7331#L196)

```go
func Background() Context
```

Background返回非nil，非空的Context。 它永远不会被取消，没有value，也没有deadline。 它通常由main函数，init和test使用，并作为传入请求的顶级Context。

## func [TODO](https://golang.org/src/context/context.go?s=7712:7731#L205)

```go
func TODO() Context
```

TODO返回非nil非空的Context。 代码应该使用context.TODO，当它不清楚使用哪个Context或者它还不可用时（因为周围的函数还没有被扩展来接受Context参数）。 TODO被静态分析工具识别，以确定Context是否在程序中正确传播。

## func  [WithCancel](https://golang.org/src/context/context.go?s=8348:8412#L220)

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
```

WithCancel返回一个新的Done channel的父亲的副本。 当返回的cancel函数被调用时，或者父Context的Done Channel  close时，返回的Context的Done Channel是关闭的，以先发生者为准。

取消这个Context将释放与它相关的资源，所以只要在这个Context中运行的操作完成，代码就应该调用cancel。

### 例子

这个例子演示了使用可取消上下文来防止goroutine泄漏。 在示例函数结束时，由gen启动的goroutine将返回而不泄漏。

```go
package main

import (
	"context"
	"fmt"
)

func main() {
	// gen generates integers in a separate goroutine and
	// sends them to the returned channel.
	// The callers of gen need to cancel the context once
	// they are done consuming generated integers not to leak
	// the internal goroutine started by gen.
	gen := func(ctx context.Context) <-chan int {
		dst := make(chan int)
		n := 1
		go func() {
			for {
				select {
				case <-ctx.Done():
					return // returning not to leak the goroutine
				case dst <- n:
					n++
				}
			}
		}()
		return dst
	}

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel() // cancel when we are finished consuming integers

	for n := range gen(ctx) {
		fmt.Println(n)
		if n == 5 {
			break
		}
	}
}
```

## func [WithDeadline](https://golang.org/src/context/context.go?s=12289:12364#L373)

```go
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
```

WithDeadline返回父Context的副本，并将deadline调整为不迟于d。 如果parent的deadline早于d，WithDeadline（parent，d）在语义上等同于parent。 当deadline到期，返回的取消函数被调用时，或者parent context的Done channel关闭时，返回的Context的Done channel是关闭的，无论哪个先发生。

取消这个上下文将释放与它相关的资源，所以只要在这个Context中运行的操作完成，代码就应该调用cancel。

### 例子

这个例子传递一个任意期限的上下文来告诉一个阻塞函数，它应该尽快放弃它的工作。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	d := time.Now().Add(50 * time.Millisecond)
	ctx, cancel := context.WithDeadline(context.Background(), d)

	// 即使ctx将过期，在任何情况下调用它的取消函数都是很好的做法。 如果不这样做，可能会使上下文及其父母的存活时间超过必要的时间。
	defer cancel()

	select {
	case <-time.After(1 * time.Second):
		fmt.Println("overslept")
	case <-ctx.Done():
		fmt.Println(ctx.Err())
	}

}
```

## func [WithTimeout](https://golang.org/src/context/context.go?s=14299:14376#L440)

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```

WithTimeout返回WithDeadline（parent，time.Now().Add（timeout））。

取消这个Context将释放与它相关的资源，所以只要在这个Context中运行的操作完成，代码应该立即调用cancel：

```go
func slowOperationWithTimeout(ctx context.Context) (Result, error) {
	ctx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
	defer cancel()  // releases resources if slowOperation completes before timeout elapses
	return slowOperation(ctx)
}
```

### 例子

这个例子传递一个超时的上下文来告诉一个阻塞函数，它应该在超时时间后放弃它的工作。

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// Pass a context with a timeout to tell a blocking function that it
	// should abandon its work after the timeout elapses.
	ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
	defer cancel()

	select {
	case <-time.After(1 * time.Second):
		fmt.Println("overslept")
	case <-ctx.Done():
		fmt.Println(ctx.Err()) // prints "context deadline exceeded"
	}

}
```


## func [WithValue](https://golang.org/src/context/context.go?s=15091:15151#L457)

```go
func WithValue(parent Context, key, val interface{}) Context
```

WithValue返回父key的副本，其中与key关联的值是val。

使用Context value仅适用于传输进程和API的请求范围数据，而不是将可选参数传递给函数。

提供的密钥必须具有可比性，不应该是字符串类型或任何其他内置类型，以避免使用上下文的包之间的冲突。 WithValue的用户应该为键定义自己的类型。 为了避免分配给接口{}时，上下文键通常具有具体类型struct {}。 或者，导出的上下文关键变量的静态类型应该是一个指针或接口。

```go
package main

import (
	"context"
	"fmt"
)

func main() {
	type favContextKey string

	f := func(ctx context.Context, k favContextKey) {
		if v := ctx.Value(k); v != nil {
			fmt.Println("found value:", v)
			return
		}
		fmt.Println("key not found:", k)
	}

	k := favContextKey("language")
	ctx := context.WithValue(context.Background(), k, "Go")

	f(ctx, k)
	f(ctx, favContextKey("color"))

}
```