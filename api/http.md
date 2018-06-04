

# Http 类库

### type [RoundTripper](https://golang.org/src/net/http/client.go?s=4367:5587#L107)

RoundTripper是一个接口，表示执行单个HTTP事务的能力，获得给定请求的响应。

RoundTripper必须是安全的，以供多个goroutine同时使用。

```go
type RoundTripper interface {
        // RoundTrip执行单个HTTP事务，为提供的请求返回响应。
        //
        // RoundTrip不应尝试解释响应。特别是，无论响应的HTTP状态码如何，
        // RoundTrip如果获得响应都必须返回err == nil。如果未能获得答复，应该保留一个非nil的错误。
        // 同样，RoundTrip不应尝试处理更高级别的协议详细信息，例如重定向，身份验证或Cookie。
        //
        // RoundTrip不应该修改请求，除了消耗和关闭请求的body。
        // RoundTrip可以在单独的goroutine中读取请求的字段。呼叫者不应该改变请求直到响应的body关闭。
        //
        // RoundTrip必须总是关闭body，包括错误，但根据实现的不同，甚至可以再在RoundTrip返回后，
        // 在单独的goroutine中执行此操作。这意味着呼叫者想要重复使用该机构以进行后续请求，
        // 必须先安排等待关闭呼叫。
        //
        // 请求的URL和标题字段必须初始化
        RoundTrip(*Request) (*Response, error)
}
```

DefaultTransport是Transport的默认实现，由DefaultClient使用。 它根据需要建立网络连接，并将它们缓存以供随后的调用重用。 它按照\$ HTTP_PROXY和\$ NO_PROXY（或\$ http_proxy和\$ no_proxy）环境变量的指示使用HTTP代理。

```go
var DefaultTransport RoundTripper = &Transport{
        Proxy: ProxyFromEnvironment,
        DialContext: (&net.Dialer{
                Timeout:   30 * time.Second,
                KeepAlive: 30 * time.Second,
                DualStack: true,
        }).DialContext,
        MaxIdleConns:          100,
        IdleConnTimeout:       90 * time.Second,
        TLSHandshakeTimeout:   10 * time.Second,
        ExpectContinueTimeout: 1 * time.Second,
}
```



## type [Transport](https://golang.org/src/net/http/transport.go?s=2973:8359#L75) 

Transport是RoundTripper的一个实现，它支持HTTP，HTTPS和HTTP代理（对于使用CONNECT的HTTP或HTTPS）。

默认情况下，Transport缓存连接以供将来重新使用。 访问多台主机时可能会留下许多open连接。 可以使用传输的CloseIdleConnections方法和MaxIdleConnsPerHost和DisableKeepAlives字段管理此行为。

Transport应该重用，而不是根据需要创建。Transport对于多个goroutines并发使用是安全的。

Transport是用于发出HTTP和HTTPS请求的低级原语。 对于高级功能（如Cookie和重定向），请参阅Client。

Transport对于HTTP URL使用HTTP / 1.1，对于HTTPS URL使用HTTP / 1.1或HTTP / 2，具体取决于服务器是否支持HTTP / 2以及如何配置Transport。 DefaultTransport支持HTTP / 2。 要在传输上明确启用HTTP / 2，请使用golang.org/x/net/http2并调用ConfigureTransport。 有关HTTP / 2的更多信息，请参阅软件包文档。

在处理HTTPS请求时，Transport将发送CONNECT请求给自己使用的代理，但传输通常不应该用于发送CONNECT请求。 也就是说，传递给RoundTrip方法的请求不应该有一个“CONNECT”方法，因为Go的HTTP / 1.x实现不支持响应正文流式传输时写入的全双工请求体。 Go的HTTP / 2实现支持全双工，但许多CONNECT代理都使用HTTP / 1.x。

```go
type Transport struct {

        //Proxy指定一个函数来返回给定请求的代理。 
        //如果该函数返回一个非nil错误，则请求会因提供的错误而中止。
        //Prxy类型由URL方案确定。 支持“http”和“socks5”。 如果该方案是空的，则假定“http”。 如果代理为nil      
        //或返回nil* URL，则不使用代理。
        Proxy func(*Request) (*url.URL, error)

        // DialContext指定用于创建未加密的TCP连接的拨号功能
        //如果DialContext为nil（不赞成使用的Dial也为nil），
        ///然后Transport使用包网络拨号.
        DialContext func(ctx context.Context, network, addr string) (net.Conn, error)

        // 拨号指定用于创建未加密的TCP连接的拨号功能.
        //
        // 弃用：使用DialContext来代替传输
        // 在不再需要拨号时立即取消拨号.
        // 如果两者都设置，则DialContext优先.
        Dial func(network, addr string) (net.Conn, error)

        //  DialTLS指定一个可选的拨号功能，用于为非代理的HTTPS请求创建TLS连接
        //
        // 如果DialTLS为nil，则使用Dial和TLSClientConfig.
        //
        // 如果设置了DialTLS，则Dial钩子不用于HTTPS请求，
        //并且TLSClientConfig和TLSHandshakeTimeout将被忽略。 假定返回的net.Conn已经通过TLS握手.
        DialTLS func(network, addr string) (net.Conn, error)

        // TLSClientConfig指定用于tls.Client的TLS配置。
        // 如果为nil, 使用默认配置.
        // 如果为非nil, 则默认情况下可能不启用HTTP/2支持.
        TLSClientConfig *tls.Config

        // TLSHandshakeTimeout指定等待TLS握手的最长时间。 nil意味着没有超时。
        TLSHandshakeTimeout time.Duration

        // DisableKeepAlives，如果为true，则阻止不同HTTP请求之间的TCP连接重用。
        DisableKeepAlives bool

        // DisableCompression（如果为true）在请求中不包含现有的Accept-Encoding值时，
        //阻止Transport请求使用“Accept-Encoding：gzip”请求标头进行压缩。 
        //如果Transport自己请求gzip并获得gzip响应，则它将在Response.Body中透明地解码。 
        //但是，如果用户明确要求gzip，它不会自动解压缩。
        DisableCompression bool

        // MaxIdleConns控制所有host的最大空闲（保持活动）连接数。 nil意味着没有限制。
        MaxIdleConns int

        // MaxIdleConnsPerHost，如果非nil，则控制最大空闲（保持活动）连接以保持每台主机。 如果为nil，
         //使用DefaultMaxIdleConnsPerHost。
        MaxIdleConnsPerHost int

        //IdleConnTimeout是空闲（保持活动）连接在关闭之前保持空闲的最长时间。
        //nil表示没有限制
        IdleConnTimeout time.Duration

        // ResponseHeaderTimeout（如果非零）指定在完全写入请求
        //（包括其主体，如果有）后等待服务器响应标头的时间量。 这一次不包括读取响应主体的时间。
        ResponseHeaderTimeout time.Duration

        //如果请求具有“Expect：100-continue”标头，则ExpectContinueTimeout（如果非0）
        //指定在完全写入请求标头之后等待服务器的第一个响应标头的时间量。 
        //0意味着没有超时，并导致正文立即发送，而无需等待服务器批准。此时间不包括发送请求标头的时间。
        ExpectContinueTimeout time.Duration

        // TLSNextProto指定Thransport在TLS NPN / ALPN协议协商后如何切换到备用协议（如HTTP / 2）。
        //如果Thransport使用非空协议名称拨号TLS连接，并且TLSNextProto包含该密钥的映射条目（例如“h2”），
        //那么将使用请求的权限调用func（例如“example.com”或“example .com：1234“）和TLS连接。 
        //该函数必须返回一个RoundTripper，然后处理该请求。 
        //如果TLSNextProto不为nil，则HTTP / 2支持不会自动启用。
        TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper

        // ProxyConnectHeader可选地指定要发送到的header
        // CONNECT请求期间的代理。
        ProxyConnectHeader Header

        //MaxResponseHeaderBytes指定了在服务器的响应头中允许多少个响应字节的限制。
        //
        // 0表示不限制
        MaxResponseHeaderBytes int64
        //包含过滤或未导出的字段
}
```

#### func (*Transport) [CancelRequest](https://golang.org/src/net/http/transport.go?s=19034:19081#L548)

```go
func (t *Transport) CancelRequest(req *Request)
```

CancelRequest通过关闭其连接来取消正在进行的请求。 只有在RoundTrip返回后才能调用CancelRequest。

已弃用：使用Request.WithContext来创建具有可取消上下文的请求。 CancelRequest无法取消HTTP / 2请求。

#### func (*Transport) [CloseIdleConnections](https://golang.org/src/net/http/transport.go?s=18347:18389#L523)

```go
func (t *Transport) CloseIdleConnections()
```

CloseIdleConnections关闭以前连接的所有连接，但是现在处于“保持活动”状态。 它不会中断当前正在使用的任何连接。

#### func (*Transport) [RegisterProtocol](https://golang.org/src/net/http/transport.go?s=17724:17792#L504)

```go
func (t *Transport) RegisterProtocol(scheme string, rt RoundTripper)
```

RegisterProtocol用scheme注册一个新的协议。 Transport将使用给定的方案将请求传递给rt。 模拟HTTP请求语义是rt的责任。

其他软件包可以使用RegisterProtocol来提供“ftp”或“file”等协议方案的实现。

如果rt.RoundTrip返回ErrSkipAltProtocol，Transport将为该请求处理RoundTrip本身，就好像该协议未注册一样。

#### func (*Transport) [RoundTrip](https://golang.org/src/net/http/transport.go?s=12586:12648#L340)

```go
func (t *Transport) RoundTrip(req *Request) (*Response, error)
```

RoundTrip实现RoundTripper接口。

对于更高级别的HTTP客户端支持（如处理Cookie和重定向），请参阅Get, Post,和Client.