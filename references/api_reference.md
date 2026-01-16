# Kratos API 参考文档

## Kratos 核心API

### App 应用

```go
import "github.com/go-kratos/kratos/v2"

// 创建应用
app := kratos.New(
    kratos.Name("helloworld"),
    kratos.Version("1.0.0"),
    kratos.Metadata(map[string]string{}),
    kratos.Server(grpcSrv, httpSrv),
    kratos.Registrar(reg),
    kratos.Logger(logger),
    kratos.TracerProvider(tp),
)

// 运行应用
if err := app.Run(); err != nil {
    panic(err)
}

// 停止应用
if err := app.Stop(context.Background()); err != nil {
    panic(err)
}
```

### Transport 传输层

#### HTTP Server

```go
import "github.com/go-kratos/kratos/v2/transport/http"

// 创建HTTP服务器
srv := http.NewServer(
    http.Address(":8000"),
    http.Network("tcp"),
    http.Timeout(time.Second),
    http.Middleware(
        recovery.Recovery(),
        tracing.Server(),
        logging.Server(logger),
    ),
)

// 注册HTTP处理器
v1.RegisterGreeterHTTPServer(srv, greeter)

// 添加路由
srv.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
})
```

#### gRPC Server

```go
import "github.com/go-kratos/kratos/v2/transport/grpc"

// 创建gRPC服务器
srv := grpc.NewServer(
    grpc.Address(":9000"),
    grpc.Network("tcp"),
    grpc.Timeout(time.Second),
    grpc.Middleware(
        recovery.Recovery(),
        tracing.Server(),
        logging.Server(logger),
    ),
)

// 注册gRPC服务
v1.RegisterGreeterServer(srv, greeter)
```

#### HTTP Client

```go
import "github.com/go-kratos/kratos/v2/transport/http"

// 创建HTTP客户端
client, err := http.NewClient(
    context.Background(),
    http.WithEndpoint("http://localhost:8000"),
    http.WithMiddleware(
        tracing.Client(),
        logging.Client(logger),
    ),
)
if err != nil {
    panic(err)
}

// 使用客户端
req, _ := http.NewRequest("GET", "/helloworld/kratos", nil)
resp, err := client.Do(req)
```

#### gRPC Client

```go
import "github.com/go-kratos/kratos/v2/transport/grpc"

// 创建gRPC客户端
conn, err := grpc.DialInsecure(
    context.Background(),
    grpc.WithEndpoint("localhost:9000"),
    grpc.WithMiddleware(
        tracing.Client(),
        logging.Client(logger),
    ),
)
if err != nil {
    panic(err)
}

// 使用客户端
client := v1.NewGreeterClient(conn)
reply, err := client.SayHello(context.Background(), &v1.HelloRequest{Name: "kratos"})
```

### Config 配置管理

```go
import "github.com/go-kratos/kratos/v2/config"

// 创建配置管理器
c := config.New(
    config.WithSource(
        file.NewSource("configs/config.yaml"),
        env.NewSource("KRATOS_"),
    ),
    config.WithDecoder(func(kv *config.KeyValue, v map[string]interface{}) error {
        // 自定义解码器
        return yaml.Unmarshal(kv.Value, v)
    }),
)

// 加载配置
if err := c.Load(); err != nil {
    panic(err)
}

// 扫描配置到结构体
var bc Bootstrap
if err := c.Scan(&bc); err != nil {
    panic(err)
}

// 监听配置变化
c.Watch("server.http", func(key string, value config.Value) {
    // 处理配置变化
})

// 关闭配置管理器
defer c.Close()
```

### Log 日志系统

```go
import "github.com/go-kratos/kratos/v2/log"

// 创建日志器
logger := log.With(log.NewStdLogger(os.Stdout),
    "ts", log.DefaultTimestamp,
    "caller", log.DefaultCaller,
    "service.id", id,
    "service.name", name,
    "service.version", version,
    "trace.id", tracing.TraceID(),
    "span.id", tracing.SpanID(),
)

// 创建日志助手
helper := log.NewHelper(logger)

// 记录日志
helper.Infof("Starting server on %s", addr)
helper.WithContext(ctx).Errorf("Failed to process request: %v", err)

// 不同级别日志
helper.Debug("debug message")
helper.Info("info message")
helper.Warn("warn message")
helper.Error("error message")

// 结构化日志
helper.WithFields(log.Fields{
    "user_id": 123,
    "action": "login",
}).Info("User logged in")
```

### Registry 服务注册与发现

```go
import "github.com/go-kratos/kratos/v2/registry"

// 服务注册
func registerService(r registry.Registrar, app *kratos.App) error {
    return r.Register(context.Background(), &registry.ServiceInstance{
        ID:        app.ID(),
        Name:      app.Name(),
        Version:   app.Version(),
        Metadata:  app.Metadata(),
        Endpoints: app.Endpoint(),
    })
}

// 服务注销
func deregisterService(r registry.Registrar, app *kratos.App) error {
    return r.Deregister(context.Background(), &registry.ServiceInstance{
        ID: app.ID(),
    })
}

// 服务发现
func discoverService(r registry.Discovery, serviceName string) ([]*registry.ServiceInstance, error) {
    return r.GetService(context.Background(), serviceName)
}

// 监听服务变化
func watchService(r registry.Discovery, serviceName string) (registry.Watcher, error) {
    return r.Watch(context.Background(), serviceName)
}
```

### Middleware 中间件

```go
import "github.com/go-kratos/kratos/v2/middleware"

// 创建中间件链
chain := middleware.Chain(
    recovery.Recovery(),
    tracing.Server(),
    logging.Server(logger),
    validate.Validator(),
    metrics.Server(),
    ratelimit.Server(),
    auth.JWTAuth(authKeyFunc),
)

// 自定义中间件
func CustomMiddleware() middleware.Middleware {
    return func(handler middleware.Handler) middleware.Handler {
        return func(ctx context.Context, req interface{}) (interface{}, error) {
            // 前置处理
            start := time.Now()

            // 调用下一个处理器
            reply, err := handler(ctx, req)

            // 后置处理
            duration := time.Since(start)
            log.Printf("Request processed in %v", duration)

            return reply, err
        }
    }
}
```

### Errors 错误处理

```go
import "github.com/go-kratos/kratos/v2/errors"

// 创建错误
err := errors.New(500, "INTERNAL_ERROR", "internal server error")

// 带元数据的错误
err = errors.New(400, "INVALID_REQUEST", "invalid request")
    .WithMetadata(map[string]string{
        "field": "name",
        "reason": "required",
    })

// 检查错误类型
if errors.Is(err, errors.BadRequest) {
    // 处理400错误
}

// 获取错误详情
code := errors.Code(err)
reason := errors.Reason(err)
message := errors.Message(err)
metadata := errors.Metadata(err)

// 转换错误
if kratosErr, ok := errors.FromError(err); ok {
    // 处理Kratos错误
}
```

### Metadata 元数据

```go
import "github.com/go-kratos/kratos/v2/metadata"

// 设置元数据
ctx := metadata.NewServerContext(context.Background(), metadata.New(map[string]string{
    "x-request-id": "12345",
    "authorization": "Bearer token",
}))

// 获取元数据
if md, ok := metadata.FromServerContext(ctx); ok {
    requestID := md.Get("x-request-id")
    authToken := md.Get("authorization")
}

// 客户端设置元数据
ctx = metadata.AppendToClientContext(ctx, "x-client-id", "client-123")

// 复制元数据
newCtx := metadata.WithContext(ctx, md)
```

## 常用工具函数

### Context 工具

```go
import "github.com/go-kratos/kratos/v2"

// 获取应用信息
appID := kratos.AppID(ctx)
appName := kratos.AppName(ctx)
appVersion := kratos.AppVersion(ctx)
appMetadata := kratos.AppMetadata(ctx)

// 检查上下文是否包含应用信息
if kratos.HasApp(ctx) {
    // 上下文包含应用信息
}
```

### Endpoint 工具

```go
import "github.com/go-kratos/kratos/v2/transport"

// 获取端点信息
endpoints := transport.Endpoints(ctx)

// 检查传输类型
if transport.IsHTTP(ctx) {
    // HTTP传输
} else if transport.IsGRPC(ctx) {
    // gRPC传输
}
```

### 验证工具

```go
import "github.com/go-kratos/kratos/v2/middleware/validate"

// 验证请求
if err := validate.Validate(req); err != nil {
    return nil, err
}

// 自定义验证器
type CustomValidator struct{}

func (v *CustomValidator) Validate() error {
    // 自定义验证逻辑
    return nil
}
```

## 配置选项

### Server 配置选项

```go
// HTTP服务器选项
http.Options{
    Network: "tcp",
    Address: ":8000",
    Timeout: time.Second * 30,
    Middleware: []middleware.Middleware{
        recovery.Recovery(),
        tracing.Server(),
    },
    Filters: []http.FilterFunc{
        // 过滤器函数
    },
    Decoder: func(r *http.Request, v interface{}) error {
        // 自定义解码器
        return json.NewDecoder(r.Body).Decode(v)
    },
    Encoder: func(w http.ResponseWriter, r *http.Request, v interface{}) error {
        // 自定义编码器
        return json.NewEncoder(w).Encode(v)
    },
    ErrorEncoder: func(w http.ResponseWriter, r *http.Request, err error) {
        // 自定义错误编码器
        code := errors.Code(err)
        w.WriteHeader(int(code))
        json.NewEncoder(w).Encode(map[string]interface{}{
            "error": err.Error(),
        })
    },
}

// gRPC服务器选项
grpc.Options{
    Network: "tcp",
    Address: ":9000",
    Timeout: time.Second * 30,
    Middleware: []middleware.Middleware{
        recovery.Recovery(),
        tracing.Server(),
    },
    UnaryInterceptor: []grpc.UnaryServerInterceptor{
        // gRPC拦截器
    },
    StreamInterceptor: []grpc.StreamServerInterceptor{
        // gRPC流拦截器
    },
}
```

### Client 配置选项

```go
// HTTP客户端选项
http.ClientOptions{
    Endpoint: "http://localhost:8000",
    Middleware: []middleware.Middleware{
        tracing.Client(),
        logging.Client(logger),
    },
    Timeout: time.Second * 30,
    UserAgent: "kratos-client/1.0",
}

// gRPC客户端选项
grpc.ClientOptions{
    Endpoint: "localhost:9000",
    Middleware: []middleware.Middleware{
        tracing.Client(),
        logging.Client(logger),
    },
    Timeout: time.Second * 30,
    Block: true,
}
```

## 最佳实践

### 错误处理模式

```go
// 统一错误处理
func handleError(c *gin.Context, err error) {
    if kratosErr, ok := errors.FromError(err); ok {
        c.JSON(int(kratosErr.Code), gin.H{
            "code":     kratosErr.Code,
            "reason":   kratosErr.Reason,
            "message":  kratosErr.Message,
            "metadata": kratosErr.Metadata,
        })
    } else {
        c.JSON(500, gin.H{
            "code":    500,
            "reason":  "INTERNAL_ERROR",
            "message": "Internal Server Error",
        })
    }
}
```

### 中间件组合

```go
// 推荐的中间件顺序
func defaultMiddleware(logger log.Logger) []middleware.Middleware {
    return []middleware.Middleware{
        // 1. 恢复
        recovery.Recovery(),
        // 2. 追踪
        tracing.Server(),
        // 3. 日志
        logging.Server(logger),
        // 4. 指标
        metrics.Server(),
        // 5. 限流
        ratelimit.Server(),
        // 6. 认证
        auth.JWTAuth(authKeyFunc),
        // 7. 验证
        validate.Validator(),
    }
}
```

### 配置管理

```go
// 配置热更新
func watchConfigChanges(c config.Config) {
    c.Watch("server.http", func(key string, value config.Value) {
        // 重新配置HTTP服务器
    })

    c.Watch("server.grpc", func(key string, value config.Value) {
        // 重新配置gRPC服务器
    })

    c.Watch("data.database", func(key string, value config.Value) {
        // 重新连接数据库
    })
}
```
