---
title: grpc-advanced
date: '2023-11-20T09:01:23+08:00'
lastmod: '2023-11-20T09:20:30+08:00'
draft: false
tags:
- golang
- grpc
categories:
- code
url: /zh-cn/2023/11/20/category/grpc-advanced/
authors:
- yehh
---

## grpc-tsl {#grpc-tsl}

### server side TLS {#server-side-tls}

有关TLS的知识结合[这个](https://www.cnblogs.com/sewain/p/14250884.html#ssltls)看体验更佳

### Interceptor {#interceptor}

有4种拦截器

```go
type UnaryClientInterceptor func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, invoker UnaryInvoker, opts ...CallOption) error

type UnaryServerInterceptor func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)

type StreamClientInterceptor func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, streamer Streamer, opts ...CallOption) (ClientStream, error)

type StreamServerInterceptor func(srv interface{}, ss ServerStream, info *StreamServerInfo, handler StreamHandler) error
```

通过WithxxxInterceptor()方法添加客户端拦截器

一般的使用场景是添加K/V元数据

下面的例子是客户端的

```go
func UnaryClientInterceptor() grpc.UnaryClientInterceptor {
    return func(ctx context.Context, method string, req, reply interface{},
        cc *grpc.ClientConn, invoker grpc.UnaryInvoker, opts ...grpc.CallOption) error {
        // 预处理(pre-processing)
        start := time.Now()
        // 获取正在运行程序的操作系统
        cos := runtime.GOOS
        // 将操作系统信息附加到传出请求
        ctx = metadata.AppendToOutgoingContext(ctx, "client-os", cos)

        // 可以看做是当前 RPC 方法，一般在拦截器中调用 invoker 能达到调用 RPC 方法的效果，当然底层也是 gRPC 在处理。
        // 调用RPC方法(invoking RPC method)
        err := invoker(ctx, method, req, reply, cc, opts...)

        // 后处理(post-processing)
        end := time.Now()
        log.Printf("RPC: %s,,client-OS: '%v' req:%v start time: %s, end time: %s, err: %v", method, cos, req, start.Format(time.RFC3339), end.Format(time.RFC3339), err)
        return err
    }
}
```

下面是服务端

```go
func UnaryServerInterceptor() grpc.UnaryServerInterceptor {
    return func(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo,
        handler grpc.UnaryHandler) (resp interface{}, err error) {
        // 预处理(pre-processing)
        start := time.Now()
        // 从传入上下文获取元数据
        md, ok := metadata.FromIncomingContext(ctx)
        if !ok {
            return nil, fmt.Errorf("couldn't parse incoming context metadata")
        }

        // 检索客户端操作系统，如果它不存在，则此值为空
        os := md.Get("client-os")
        // 获取客户端IP地址
        ip, err := getClientIP(ctx)
        if err != nil {
            return nil, err
        }

        // RPC 方法真正执行的逻辑
        // 调用RPC方法(invoking RPC method)
        m, err := handler(ctx, req)
        end := time.Now()
        // 记录请求参数 耗时 错误信息等数据
        // 后处理(post-processing)
        log.Printf("RPC: %s,client-OS: '%v' and IP: '%v' req:%v start time: %s, end time: %s, err: %v", info.FullMethod, os, ip, req, start.Format(time.RFC3339), end.Format(time.RFC3339), err)
        return m, err
    }
}

// GetClientIP检查上下文以检索客户机的ip地址
func getClientIP(ctx context.Context) (string, error) {
    p, ok := peer.FromContext(ctx)
    if !ok {
        return "", fmt.Errorf("couldn't parse client IP address")
    }
    return p.Addr.String(), nil
}
```

#### 流式拦截 {#流式拦截}

流程差不多，3个阶段

```text
1）预处理 (pre-processing)
2）调用 RPC 方法 (invoking RPC method)
3）后处理 (post-processing)
```

但是在2，3步的时候有所不同，比如是调用streamer而不是invoker，而且是要直接放回streamer对象。 拦截流的话需要重写streamer的SendMsg和RecvMsg方法

```go
func StreamClientInterceptor() grpc.StreamClientInterceptor {
    return func(ctx context.Context, desc *grpc.StreamDesc, cc *grpc.ClientConn,
        method string, streamer grpc.Streamer, opts ...grpc.CallOption) (grpc.ClientStream, error) {
        log.Printf("opening client streaming to the server method: %v", method)
        // 调用Streamer函数，获得ClientStream
        stream, err := streamer(ctx, desc, cc, method)
        return newStreamClient(stream), err
    }
}

// 嵌入式 streamClient 允许我们访问SendMsg和RecvMsg函数
type streamClient struct {
    grpc.ClientStream
}

// 对ClientStream进行包装
func newStreamClient(c grpc.ClientStream) grpc.ClientStream {
    return &streamClient{c}
}

// RecvMsg从流中接收消息
func (e *streamClient) RecvMsg(m interface{}) error {
    // 在这里，我们可以对接收到的消息执行额外的逻辑，例如
    // 验证
    log.Printf("Receive a message (Type: %T) at %v", m, time.Now().Format(time.RFC3339))
    if err := e.ClientStream.RecvMsg(m); err != nil {
        return err
    }
    return nil
}

// RecvMsg从流中接收消息
func (e *streamClient) SendMsg(m interface{}) error {
    // 在这里，我们可以对接收到的消息执行额外的逻辑，例如
    // 验证
    log.Printf("Send a message (Type: %T) at %v", m, time.Now().Format(time.RFC3339))
    if err := e.ClientStream.SendMsg(m); err != nil {
        return err
    }
    return nil
}
```

```go
conn, err := grpc.Dial(":"+PORT, grpc.WithInsecure(), grpc.WithStreamInterceptor(Interceptor.StreamClientInterceptor()))
if err != nil {
    log.Fatalf("grpc.Dial err: %v", err)
}
defer conn.Close()
```

### 自定义身份验证 {#自定义身份验证}

## 参考 {#参考}

1.  [grpc进阶](https://learnku.com/articles/73105)
