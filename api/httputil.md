# HttpUtil

软件包httputil提供HTTP实用程序功能，补充了net / http软件包中较常见的功能。

## 变量

```go
var (
        // 弃用: No longer used.
        ErrPersistEOF = &http.ProtocolError{ErrorString: "persistent connection closed"}

        // 弃用: No longer used.
        ErrClosed = &http.ProtocolError{ErrorString: "connection closed by user"}

        // 弃用: No longer used.
        ErrPipeline = &http.ProtocolError{ErrorString: "pipeline error"}
)
```

使用太长的行读取格式不正确的分块数据时，会返回ErrLineTooLong。

```go
var ErrLineTooLong = internal.ErrLineTooLong
```

## func [DumpRequest](https://golang.org/src/net/http/httputil/dump.go?s=5638:5700#L181)

```go
func DumpRequest(req *http.Request, body bool) ([]byte, error)
```

DumpRequest以HTTP / 1.x wire representation形式返回给定的请求。 它只能被服务器用来调试客户端请求。 返回的representation只是一个近似值; 解析为http.Request时，初始请求的某些细节会丢失。 特别是head字段名称的顺序和大小写会丢失。 多值头中值的顺序保持不变。 HTTP / 2请求以HTTP / 1.x形式转储，而不是以其原始二进制表示。

如果body正确，DumpRequest也会返回body。 为此，它使用req.Body，然后用一个新的产生相同字节的io.ReadCloser替换它。 如果DumpRequest返回err，则req的状态是未定义的。

http.Request.Write的文档详细说明了哪些req字段包含在转储中。

#### 例子

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"net/http/httptest"
	"net/http/httputil"
	"strings"
)

func main() {
	ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		dump, err := httputil.DumpRequest(r, true)
		if err != nil {
			http.Error(w, fmt.Sprint(err), http.StatusInternalServerError)
			return
		}

		fmt.Fprintf(w, "%q", dump)
	}))
	defer ts.Close()

	const body = "Go is a general-purpose language designed with systems programming in mind."
	req, err := http.NewRequest("POST", ts.URL, strings.NewReader(body))
	if err != nil {
		log.Fatal(err)
	}
	req.Host = "www.example.org"
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", b)

}
```

## func [DumpRequestOut](https://golang.org/src/net/http/httputil/dump.go?s=1848:1913#L56)

```go
func DumpRequestOut(req *http.Request, body bool) ([]byte, error)
```

DumpRequestOut就像DumpRequest，但是用于传出客户端请求。 它包括标准http.Transport添加的任何head，例如User-Agent。

### 例子

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"net/http/httputil"
	"strings"
)

func main() {
	const body = "Go is a general-purpose language designed with systems programming in mind."
	req, err := http.NewRequest("PUT", "http://www.example.org", strings.NewReader(body))
	if err != nil {
		log.Fatal(err)
	}

	dump, err := httputil.DumpRequestOut(req, true)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%q", dump)

}
```

## func [DumpResponse](https://golang.org/src/net/http/httputil/dump.go?s=8166:8231#L271)

```go
func DumpResponse(resp *http.Response, body bool) ([]byte, error)
```

DumpResponse与DumpRequest类似，但它是转存储储response。

### 例子

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"net/http/httptest"
	"net/http/httputil"
)

func main() {
	const body = "Go is a general-purpose language designed with systems programming in mind."
	ts := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		w.Header().Set("Date", "Wed, 19 Jul 1972 19:00:00 GMT")
		fmt.Fprintln(w, body)
	}))
	defer ts.Close()

	resp, err := http.Get(ts.URL)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()

	dump, err := httputil.DumpResponse(resp, true)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%q", dump)

}
```

## func [NewChunkedReader](https://golang.org/src/net/http/httputil/httputil.go?s=688:732#L10)

```go
func NewChunkedWriter(w io.Writer) io.WriteCloser
```

NewChunkedWriter返回一个新的chunkedWriter，在写入w之前将写入转换为HTTP“分块(chunked)”格式。 关闭返回的chunkedWriter将发送标记流结束的最终0长度块。

普通应用程序不需要NewChunkedWriter。 如果处理程序未设置Content-Length标头，则http包会自动添加分块。 在处理程序中使用NewChunkedWriter会导致双重组块或使用Content-Length长度分块，这两者都是错误的。

## type [BufferPool](https://golang.org/src/net/http/httputil/reverseproxy.go?s=1887:1943#L56)

BufferPool是一个获取和返回io.CopyBuffer使用的临时字节片的接口。

```go
type BufferPool interface {
        Get() []byte
        Put([]byte)
}
```

## type [ClientConn](https://golang.org/src/net/http/httputil/persist.go?s=5973:6341#L220)

ClientConn是Go早期HTTP实现的产物。 它是Go的当前HTTP堆栈的低级，旧的和未使用的。 我们应该在Go 1之前删除它。

弃用：在包net / http中使用客户端或传输。

```go
type ClientConn struct {
        // contains filtered or unexported fields
}
```

## func (*ClientConn) [Close](https://golang.org/src/net/http/httputil/persist.go?s=7765:7800#L276)

```go
func (cc *ClientConn) Close() error
```

关闭通话劫持，然后关闭底层连接。

## func (*ClientConn) [Do](https://golang.org/src/net/http/httputil/persist.go?s=11092:11159#L415)

```go
func (cc *ClientConn) Do(req *http.Request) (*http.Response, error)
```

Do是写请求并读取响应的便利方法。

## func (*ClientConn) [Hijack](https://golang.org/src/net/http/httputil/persist.go?s=7541:7601#L265)

```go
func (cc *ClientConn) Hijack() (c net.Conn, r *bufio.Reader)
```

劫持分离ClientConn并返回底层连接以及可能留下数据的读取端bufio。 可以在使用或Read之前调用劫持标志保持活动逻辑的结束。 在读取或写入过程中，用户不应该调用劫持。

## func (*ClientConn) [Pending](https://golang.org/src/net/http/httputil/persist.go?s=9365:9400#L343)

```go
func (cc *ClientConn) Pending() int
```

挂起返回已经在连接上发送的未应答请求的数量。

## func (*ClientConn) [Read](https://golang.org/src/net/http/httputil/persist.go?s=9745:9823#L353)

```go
func (cc *ClientConn) Read(req *http.Request) (resp *http.Response, err error)
```

Read读取下一个响应。 有效的响应可能会与ErrPersistEOF一起返回，这意味着远程请求这是最后一次请求服务。 Read可以与Write同时调用，但不能与另一个Read一起调用。

## func (*ClientConn) [Write](https://golang.org/src/net/http/httputil/persist.go?s=8265:8317#L289)

```go
func (cc *ClientConn) Write(req *http.Request) error
```

Write写入请求。 如果连接已经以HTTP Keepalive意义关闭，则返回ErrPersistEOF错误。 如果req.Close等于true，则在此请求和对端服务器被通知后，保持连接在逻辑上关闭。 ErrUnexpectedEOF指示远程关闭了底层TCP连接，通常认为该连接处于优雅关闭状态。

## type [ReverseProxy](https://golang.org/src/net/http/httputil/reverseproxy.go?s=575:1776#L18)

ReverseProxy是一个HTTP处理程序，它接收传入的请求并将其发送到另一个服务器，将响应代理回客户端。

```go
type ReverseProxy struct {
        // Director必须是一个函数，它将请求修改为使用Transfor发送的新请求。 
        // 其响应后被复制回未修改的原始客户端。 Director在返回后不得访问提供的请求。
        Director func(*http.Request)

        //用于执行代理请求的传输。 如果为零，则使用http.DefaultTransport。
        Transport http.RoundTripper

        // FlushInterval指定在复制响应主体时刷新到客户端的刷新间隔。 如果为nil，则不进行周期性刷新。
        FlushInterval time.Duration

        // ErrorLog指定一个可选的记录器，用于尝试代理请求时发生的错误。 
        // 如果为nil，则日志记录将通过日志包的标准记录器转到os.Stderr.
        ErrorLog *log.Logger

        // BufferPool可以选择指定一个缓冲池来获取字节片，以便在复制HTTP响应体时由io.CopyBuffer使用。
        BufferPool BufferPool

        //ModifyResponse是一个可选函数，用于修改来自后端的响应。
        //如果它返回一个错误，代理将返回一个StatusBadGateway错误。
        ModifyResponse func(*http.Response) error
}

```

### 例子

```go
package main

import (
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"net/http/httptest"
	"net/http/httputil"
	"net/url"
)

func main() {
	backendServer := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "this call was relayed by the reverse proxy")
	}))
	defer backendServer.Close()

	rpURL, err := url.Parse(backendServer.URL)
	if err != nil {
		log.Fatal(err)
	}
	frontendProxy := httptest.NewServer(httputil.NewSingleHostReverseProxy(rpURL))
	defer frontendProxy.Close()

	resp, err := http.Get(frontendProxy.URL)
	if err != nil {
		log.Fatal(err)
	}

	b, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Fatal(err)
	}

	fmt.Printf("%s", b)

}
```

###  func [NewSingleHostReverseProxy](https://golang.org/src/net/http/httputil/reverseproxy.go?s=2588:2649#L80)

```go
func NewSingleHostReverseProxy(target *url.URL) *ReverseProxy
```

NewSingleHostReverseProxy返回一个新的ReverseProxy，它将URL路由到目标中提供的方案，主机和基本路径。 如果目标路径为“/ base”，并且传入请求为“/ dir”，则目标请求将为/ base / dir。 NewSingleHostReverseProxy不重写Host头。 要重写Host头文件，请使用ReverseProxy直接使用自定义Director策略。

### func (*ReverseProxy) [ServeHTTP](https://golang.org/src/net/http/httputil/reverseproxy.go?s=4028:4103#L131)

```go
func (p *ReverseProxy) ServeHTTP(rw http.ResponseWriter, req *http.Request)
```

### 