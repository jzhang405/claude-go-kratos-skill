---
name: go-kratos
description: Comprehensive skill for working with Kratos microservice framework in Go. This skill should be used when users need help with microservice development, including service creation, gRPC/HTTP APIs, service discovery, configuration management, and distributed tracing.
---

# Go Kratos 微服务框架

## 概述

Kratos 是一套轻量级 Go 微服务框架，包含大量微服务相关框架及工具。名字来源于希腊神话中的战神奎托斯。

### 设计原则

- **简单**：不过度设计，代码平实简单
- **通用**：通用业务开发所需要的基础库功能
- **高效**：提高业务迭代的效率
- **稳定**：基础库可测试性高，覆盖率高，有线上实践安全可靠
- **高性能**：性能高，但不特定为了性能做 hack 优化
- **扩展性**：良好的接口设计，支持扩展实现
- **容错性**：为失败设计，鲁棒性高
- **工具链**：包含大量工具链

### 核心特性

- **APIs**：协议通信以 HTTP/gRPC 为基础，通过 Protobuf 定义
- **Errors**：通过 Protobuf 的 Enum 作为错误码定义
- **Metadata**：在协议通信 HTTP/gRPC 中，通过 Middleware 规范化服务元信息传递
- **Config**：支持多数据源方式，进行配置合并铺平，支持动态配置
- **Logger**：标准日志接口，可方便集成三方 log 库
- **Metrics**：统一指标接口，默认集成 Prometheus
- **Tracing**：遵循 OpenTelemetry 规范定义，实现微服务链路追踪
- **Encoding**：支持 Accept 和 Content-Type 进行自动选择内容编码
- **Transport**：通用的 HTTP/gRPC 传输层，实现统一的 Middleware 插件支持
- **Registry**：实现统一注册中心接口，可插件化对接各种注册中心

## 项目结构 (kratos-layout)

### 标准目录结构

```
project/
├── api/                          # API 定义
│   └── helloworld/
│       └── v1/
│           ├── error_reason.pb.go        # 错误定义生成代码
│           ├── error_reason.proto        # 错误定义
│           ├── error_reason.swagger.json #
│           ├── greeter.pb.go             # Protobuf 消息定义
│           ├── greeter.proto             # Proto 定义
│           ├── greeter.swagger.json      #
│           ├── greeter_grpc.pb.go        # gRPC 服务定义
│           └── greeter_http.pb.go        # HTTP Handler
├── cmd/                           # 应用入口
│   └── server/
│       ├── main.go
│       ├── wire.go                # Wire 依赖注入配置
│       └── wire_gen.go            # Wire 自动生成代码
├── configs/                       # 配置文件
│   └── config.yaml
├── internal/                      # 内部代码（不对外暴露）
│   ├── biz/                       # 业务逻辑层（DDD Domain）
│   │   ├── README.md
│   │   ├── biz.go
│   │   └── greeter.go
│   ├── conf/                      # 配置结构定义（proto 生成）
│   │   ├── conf.pb.go
│   │   └── conf.proto
│   ├── data/                      # 数据访问层（DDD Repo 实现）
│   │   ├── README.md
│   │   ├── data.go
│   │   └── greeter.go
│   ├── server/                    # Server 层（HTTP/gRPC 实例）
│   │   ├── grpc.go
│   │   ├── http.go
│   │   └── server.go
│   └── service/                   # Service 层（DDD Application）
│       ├── README.md
│       ├── greeter.go
│       └── service.go
├── third_party/                   # 第三方 proto 依赖
└── go.mod
```

### 各层职责

| 层级 | 职责 | 说明 |
|------|------|------|
| **api** | API 定义 | proto 文件及生成代码 |
| **cmd** | 入口 | 应用启动入口，Wire 注入点 |
| **internal/biz** | 业务逻辑 | 领域模型、业务规则、Repo 接口定义 |
| **internal/data** | 数据访问 | 实现 biz 定义的 Repo 接口 |
| **internal/service** | 服务层 | DTO 到 DO 转换，协调 biz 层 |
| **internal/server** | 传输层 | HTTP/gRPC Server 创建配置 |

## CLI 工具

### 安装

```bash
# 安装 CLI 工具
go install github.com/go-kratos/kratos/cmd/kratos/v2@latest

# 验证安装
kratos -v

# 升级到最新版本
kratos upgrade
```

### 命令列表

```bash
kratos new <project-name>              # 创建新项目
kratos proto add <path>                # 添加 proto 模板
kratos proto client <path>             # 生成客户端代码
kratos proto server <path>             # 生成服务端代码
kratos run                             # 运行项目
kratos changelog                       # 查看更新日志
```

### 创建项目

```bash
# 使用默认模板创建
kratos new helloworld

# 国内使用 Gitee 镜像
kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git

# 指定分支
kratos new helloworld -b main

# 大仓模式（共用 go.mod）
kratos new helloworld
cd helloworld
kratos new app/user --nomod
```

### Proto 相关命令

```bash
# 添加 proto 文件
kratos proto add api/helloworld/v1/demo.proto

# 生成客户端代码
kratos proto client api/helloworld/v1/demo.proto

# 生成服务端代码
kratos proto server api/helloworld/v1/demo.proto -t internal/service
```

## 快速开始

### 环境准备

```bash
# 安装 Go (1.21+)
# 安装 protoc
brew install protobuf

# 安装 protoc 插件
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
go install github.com/go-kratos/kratos/cmd/protoc-gen-go-http/v2@latest
go install github.com/google/gnostic/cmd/protoc-gen-openapi@latest
go install github.com/google/wire/cmd/wire@latest

# 启用 Go Modules
go env -w GO111MODULE=on
```

### 创建并运行项目

```bash
# 1. 创建项目
kratos new helloworld
cd helloworld

# 2. 安装依赖
go mod download
go get github.com/google/wire/cmd/wire@latest

# 3. 生成代码
go generate ./...

# 4. 运行
kratos run
```

### 测试接口

```bash
# HTTP API
curl 'http://127.0.0.1:8000/helloworld/kratos'
# {"message":"Hello kratos"}

# 错误响应
curl 'http://127.0.0.1:8000/helloworld/error'
# {"code":404,"reason":"USER_NOT_FOUND","message":"user not found: error"}
```

## API 定义

### Proto 定义示例

```protobuf
syntax = "proto3";
package helloworld.v1;
import "google/api/annotations.proto";
option go_package = "github.com/go-kratos/service-layout/api/helloworld/v1;v1";

// Greeter 服务定义
service Greeter {
  // 发送问候
  rpc SayHello (HelloRequest) returns (HelloReply) {
    option (google.api.http) = {
      get: "/helloworld/{name}"
      additional_bindings {
        post: "/v1/greeter/say_hello"
        body: "*"
      }
    };
  }
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

### 生成代码

```bash
# 生成 proto 模板
kratos proto add api/helloworld/v1/greeter.proto

# 生成客户端源码
kratos proto client api/helloworld/v1/greeter.proto

# 生成服务端源码
kratos proto server api/helloworld/v1/greeter.proto -t internal/service
```

### 注册服务

**HTTP API**：

```go
import "github.com/go-kratos/kratos/v2/transport/http"

greeter := &GreeterService{}
srv := http.NewServer(http.Address(":9000"))
v1.RegisterGreeterHTTPServer(srv, greeter)
```

**gRPC API**：

```go
import "github.com/go-kratos/kratos/v2/transport/grpc"

greeter := &GreeterService{}
srv := grpc.NewServer(grpc.Address(":9000"))
v1.RegisterGreeterServer(srv, greeter)
```

## 配置管理

### 支持的配置源

Kratos config 组件支持多种配置源：
- **本地文件**：`file` - 从文件系统加载
- **环境变量**：`env` - 从环境变量读取
- **配置中心**：`apollo`, `consul`, `etcd`, `kubernetes`, `nacos`, `polaris`

### 使用配置

```go
import (
    "github.com/go-kratos/kratos/v2/config"
    "github.com/go-kratos/kratos/v2/config/file"
)

path := "configs/config.yaml"
c := config.New(
    config.WithSource(
        file.NewSource(path),
    ),
)

// 加载配置
if err := c.Load(); err != nil {
    panic(err)
}

// 读取配置到结构体
var v struct {
    Service struct {
        Name    string `json:"name"`
        Version string `json:"version"`
    } `json:"service"`
}
if err := c.Scan(&v); err != nil {
    panic(err)
}

// 监听配置变更
if err := c.Watch("service.name", func(key string, value config.Value) {
    fmt.Printf("config changed: %s = %v\n", key, value)
}); err != nil {
    log.Error(err)
}
```

### 环境变量

```go
c := config.New(
    config.WithSource(
        env.NewSource("KRATOS_"),  // 前缀
        file.NewSource(path),
    ),
)
```

配置文件中使用占位符：

```yaml
service:
  name: "${service.name}"
  port: "${PORT:8080}"
```

### Protobuf 定义配置

```protobuf
syntax = "proto3";
package kratos.api;
option go_package = "github.com/go-kratos/kratos-layout/internal/conf;conf";
import "google/protobuf/duration.proto";

message Bootstrap {
  Server server = 1;
}

message Server {
  message HTTP {
    string network = 1;
    string addr = 2;
    google.protobuf.Duration timeout = 3;
  }
  message GRPC {
    string network = 1;
    string addr = 2;
    google.protobuf.Duration timeout = 3;
  }
  HTTP http = 1;
  GRPC grpc = 2;
}
```

生成命令：`make config`

## 错误处理

### 错误定义 (Proto)

```protobuf
syntax = "proto3";
package api.kratos.v1;
import "errors/errors.proto";

option go_package = "kratos/api/helloworld;helloworld";

enum ErrorReason {
  option (errors.default_code) = 500;
  USER_NOT_FOUND = 0 [(errors.code) = 404];
  CONTENT_MISSING = 1 [(errors.code) = 400];
}
```

### 生成错误代码

```bash
# 安装工具
go install github.com/go-kratos/kratos/cmd/protoc-gen-go-errors/v2@latest

# 生成错误代码
protoc --proto_path=. \
       --proto_path=./third_party \
       --go_out=paths=source_relative:. \
       --go-errors_out=paths=source_relative:. \
       $(API_PROTO_FILES)

# 或使用 make
make errors
```

### 生成的代码

```go
package helloworld

import "github.com/go-kratos/kratos/v2/errors"

func IsUserNotFound(err error) bool {
    if err == nil {
        return false
    }
    e := errors.FromError(err)
    return e.Reason == ErrorReason_USER_NOT_FOUND.String() && e.Code == 404
}

func ErrorUserNotFound(format string, args ...interface{}) *errors.Error {
    return errors.New(404, ErrorReason_USER_NOT_FOUND.String(), fmt.Sprintf(format, args...))
}
```

### 使用错误

```go
// 响应错误
errors.New(500, "USER_NAME_EMPTY", "user name is empty")
api.ErrorUserNotFound("user %s not found", "kratos")

// 传递 metadata
err := errors.New(500, "USER_NAME_EMPTY", "user name is empty")
err = err.WithMetadata(map[string]string{"foo": "bar"})

// 错误断言
if errors.Is(err, errors.BadRequest("USER_NAME_EMPTY", "")) {
    // do something
}
if helloworld.IsUserNotFound(err) {
    // do something
}
```

## 日志

### Logger 接口

```go
type Logger interface {
    Log(level Level, keyvals ...interface{}) error
}

// 日志等级
log.LevelDebug
log.LevelInfo
log.LevelWarn
log.LevelError
log.LevelFatal
```

### 使用 Helper

```go
import "github.com/go-kratos/kratos/v2/log"

// 使用默认 Logger
h := log.NewHelper(log.DefaultLogger)

// 打印日志
h.Debug("Are you OK?")
h.Info("42 is the answer")
h.Warn("We are under attack!")
h.Error("Houston, we have a problem.")
h.Fatal("So Long, and Thanks for All the Fish.")

// 格式化打印
h.Infof("%d is the answer", 233)
h.Errorf("%s, we have a problem.", "Master Shifu")

// 带字段打印
h.Infow("user_id", 123, "action", "login")
```

### 全局日志

```go
import "github.com/go-kratos/kratos/v2/log"

log.Info("info")
log.Warn("warn")

// 设置全局 Logger
log.SetLogger(logger)
```

### Valuer 设置全局字段

```go
logger = log.With(logger,
    "ts", log.DefaultTimestamp,
    "caller", log.DefaultCaller,
    "service.id", id,
    "service.name", Name,
    "service.version", Version,
    "trace_id", tracing.TraceID(),
    "span_id", tracing.SpanID(),
)
```

### Filter 日志过滤

```go
h := log.NewHelper(
    log.NewFilter(logger,
        log.FilterLevel(log.LevelError),
        log.FilterKey("password"),
        log.FilterValue("secret"),
        log.FilterFunc(func(level log.Level, keyvals ...interface{}) bool {
            // 自定义过滤逻辑
            return false
        }),
    ),
)
```

### 适配第三方日志库

```go
import (
    "github.com/go-kratos/kratos/contrib/log/zap/v2"
    "go.uber.org/zap"
)

f, _ := os.OpenFile("test.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0666)
writeSyncer := zapcore.AddSync(f)
encoder := zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)
z := zap.New(core)
logger := kratoszap.NewLogger(z)
log.SetLogger(logger)
```

## gRPC Transport

### Server 配置

```go
import "github.com/go-kratos/kratos/v2/transport/grpc"

srv := grpc.NewServer(
    grpc.Network("tcp"),
    grpc.Address(":9000"),
    grpc.Timeout(1 * time.Second),
    grpc.Logger(logger),
    grpc.Middleware(
        recovery.Recovery(),
        tracing.Server(),
        logging.Server(),
    ),
    grpc.TLSConfig(tlsConfig),
    grpc.UnaryInterceptor(unaryInts...),
    grpc.StreamInterceptor(streamInts...),
)
```

### Client 配置

```go
import "github.com/go-kratos/kratos/v2/transport/grpc"

conn, err := grpc.Dial(
    context.Background(),
    grpc.WithEndpoint("127.0.0.1:9000"),
    grpc.WithTimeout(3600 * time.Second),
    grpc.WithMiddleware(
        recovery.Recovery(),
        validate.Validator(),
    ),
    grpc.WithDiscovery(r),           // 服务发现
    grpc.WithTLSConfig(tlsConfig),
    grpc.WithHealthCheck(true),
    grpc.WithNodeFilter(filter),
)

// 使用服务发现
conn, err := grpc.DialInsecure(
    context.Background(),
    grpc.WithEndpoint("discovery:///helloworld"),
    grpc.WithDiscovery(r),
)
```

## HTTP Transport

### Server 配置

```go
import "github.com/go-kratos/kratos/v2/transport/http"

srv := http.NewServer(
    http.Network("tcp"),
    http.Address(":8000"),
    http.Timeout(1 * time.Second),
    http.Logger(logger),
    http.Middleware(
        recovery.Recovery(),
        tracing.Server(),
        logging.Server(),
    ),
    http.Filter(filters...),           // HTTP 原生 Filter
    http.RequestDecoder(decoders...),  // 自定义请求解码
    http.ResponseEncoder(encoder),     // 自定义响应编码
    http.ErrorEncoder(errorEncoder),   // 自定义错误编码
    http.TLSConfig(tlsConfig),
    http.StrictSlash(true),
)
```

### Client 配置

```go
conn, err := http.NewClient(
    context.Background(),
    http.WithEndpoint("127.0.0.1:8000"),
    http.WithTimeout(3600 * time.Second),
    http.WithMiddleware(
        recovery.Recovery(),
    ),
    http.WithDiscovery(r),             // 服务发现
    http.WithTransport(trans),         // 自定义 RoundTripper
    http.WithUserAgent("kratos-client"),
    http.WithBalancer(balancer),
    http.WithBlock(),                  // 阻塞直到发现节点
)

// 使用服务发现
conn, err := http.NewClient(
    context.Background(),
    http.WithEndpoint("discovery:///helloworld"),
    http.WithDiscovery(r),
)
```

### Router 使用

```go
// 创建子路由
r := s.Route("/v1")
r.GET("/helloworld/{name}", _Greeter_SayHello0_HTTP_Handler(srv))

// 标准 Handler
s.Handle("/path", handler)
s.HandlePrefix("/prefix/", handler)
```

## 中间件

### 内置中间件

| 中间件 | 用途 |
|--------|------|
| `recovery` | 异常恢复 |
| `tracing` | 链路追踪 |
| `logging` | 请求日志 |
| `validate` | 参数校验 |
| `metadata` | 元信息传递 |
| `auth` | JWT 认证 |
| `ratelimit` | 限流 |
| `circuitbreaker` | 熔断器 |
| `metrics` | 监控指标 |

### 使用中间件

```go
// Server 端
grpcSrv := grpc.NewServer(
    grpc.Middleware(
        recovery.Recovery(),
        tracing.Server(),
        logging.Server(),
    ),
)

httpSrv := http.NewServer(
    http.Middleware(
        recovery.Recovery(),
        tracing.Server(),
    ),
)

// Client 端
conn, err := grpc.DialInsecure(
    context.Background(),
    grpc.WithMiddleware(
        recovery.Recovery(),
    ),
)
```

### 异常恢复 (Recovery)

```go
import "github.com/go-kratos/kratos/v2/middleware/recovery"

recovery.Recovery()
```

### 链路追踪 (Tracing)

```go
import "github.com/go-kratos/kratos/v2/middleware/tracing"

tracing.Server()
tracing.Client()
```

### 参数校验 (Validate)

```go
import "github.com/go-kratos/kratos/v2/middleware/validate"

validate.Validator()
```

### 限流器 (RateLimit)

```go
import "github.com/go-kratos/kratos/v2/middleware/ratelimit"

ratelimit.Server()
ratelimit.Client()
```

### 熔断器 (CircuitBreaker)

```go
import "github.com/go-kratos/kratos/v2/middleware/circuitbreaker"

circuitbreaker.Server()
circuitbreaker.Client()
```

### JWT 认证 (Auth)

```go
import "github.com/go-kratos/kratos/v2/middleware/auth"

auth.Server(jwtKey)
auth.Client(jwtKey)
```

### 自定义中间件

```go
import (
    "context"
    "github.com/go-kratos/kratos/v2/middleware"
    "github.com/go-kratos/kratos/v2/transport"
)

func Middleware1() middleware.Middleware {
    return func(handler middleware.Handler) middleware.Handler {
        return func(ctx context.Context, req interface{}) (reply interface{}, err error) {
            if tr, ok := transport.FromServerContext(ctx); ok {
                // 进入时处理
                defer func() {
                    // 退出时处理
                }()
            }
            return handler(ctx, req)
        }
    }
}
```

### 定制中间件（路由匹配）

```go
import "github.com/go-kratos/kratos/v2/middleware/selector"

// HTTP Server
http.Middleware(
    selector.Server(
        recovery.Recovery(),
        tracing.Server(),
        customMiddleware,
    ).Path("/hello.Update/UpdateUser").
     Regex(`/test.hello/Get[0-9]+`).
     Prefix("/kratos.", "/go-kratos.").
     Match(func(ctx context.Context, operation string) bool {
         return strings.HasPrefix(operation, "/go-kratos.dev")
     }).Build(),
)

// gRPC Server
grpc.Middleware(
    selector.Server(
        recovery.Recovery(),
        tracing.Server(),
    ).Path("/helloworld.Greeter/SayHello").Build(),
)
```

**注意**：定制中间件通过 operation 匹配，operation 格式为 `/包名.服务名/方法名`

## 元信息传递

### 注册中间件

```go
// Server
grpcSrv := grpc.NewServer(
    grpc.Middleware(
        metadata.Server(),
    ),
)
httpSrv := http.NewServer(
    http.Middleware(
        metadata.Server(),
    ),
)

// Client
conn, _ := grpc.DialInsecure(
    context.Background(),
    grpc.WithMiddleware(
        metadata.Client(),
    ),
)
```

### 获取/设置元信息

```go
import "github.com/go-kratos/kratos/v2/metadata"

// 获取
if md, ok := metadata.FromServerContext(ctx); ok {
    extra := md.Get("x-md-global-extra")
}

// 设置
ctx = metadata.AppendToClientContext(ctx, "x-md-global-extra", "2233")
```

### 元信息格式规范

- `x-md-global-xxx` - 全局传递
- `x-md-local-xxx` - 局部传递

## 服务注册与发现

### 支持的注册中心

- `consul`
- `discovery`
- `etcd`
- `kubernetes`
- `nacos`
- `polaris`
- `zookeeper`

### 服务注册

```go
import (
    "github.com/go-kratos/kratos/v2"
    consul "github.com/go-kratos/kratos/contrib/registry/consul/v2"
    "github.com/hashicorp/consul/api"
)

// Consul
client, _ := api.NewClient(api.DefaultConfig())
reg := consul.New(client)

app := kratos.New(
    kratos.Name(Name),
    kratos.Version(Version),
    kratos.Metadata(map[string]string{}),
    kratos.Server(hs, gs),
    kratos.Registrar(reg),
)

// Etcd
import etcd "github.com/go-kratos/kratos/contrib/registry/etcd/v2"
client, _ := clientv3.New(clientv3.Config{
    Endpoints: []string{"127.0.0.1:2379"},
})
reg := etcd.New(client)
```

### 服务发现 (gRPC)

```go
import (
    consul "github.com/go-kratos/kratos/contrib/registry/consul/v2"
    "github.com/go-kratos/kratos/v2/transport/grpc"
    "github.com/hashicorp/consul/api"
)

client, _ := api.NewClient(api.DefaultConfig())
dis := consul.New(client)

endpoint := "discovery:///provider"  // 格式: discovery://<authority>/<serviceName>
conn, err := grpc.Dial(
    context.Background(),
    grpc.WithEndpoint(endpoint),
    grpc.WithDiscovery(dis),
)
```

## 路由与负载均衡

### 支持的负载均衡算法

- `wrr` - Weighted Round Robin（默认）
- `p2c` - Power of two choices
- `random` - Random

### 使用示例

```go
import (
    "github.com/go-kratos/kratos/v2/selector/wrr"
    "github.com/go-kratos/kratos/v2/selector/filter"
)

// HTTP Client
filter := filter.Version("2.0.0")
selector.SetGlobalSelector(wrr.NewBuilder())

hConn, err := http.NewClient(
    context.Background(),
    http.WithEndpoint("discovery:///helloworld"),
    http.WithDiscovery(r),
    http.WithNodeFilter(filter),
)

// gRPC Client
filter := filter.Version("2.0.0")
selector.SetGlobalSelector(wrr.NewBuilder())

conn, err := grpc.DialInsecure(
    context.Background(),
    grpc.WithEndpoint("discovery:///helloworld"),
    grpc.WithDiscovery(r),
    grpc.WithNodeFilter(filter),
)
```

### NodeFilter 匹配规则

```go
filter.Version("2.0.0")              // 版本匹配
filter.Address("127.0.0.1:8000")     // 地址匹配
filter.Labels(map[string]string{"zone": "us-east-1"})  // 标签匹配

// 自定义匹配
selector.NodeFilter(func(node selector.Node) bool {
    return node.Address() == "127.0.0.1:8000"
})
```

## 序列化

### 支持的格式

- `json`
- `protobuf`
- `xml`
- `yaml`
- `form`

### Codec 接口

```go
type Codec interface {
    Marshal(v interface{}) ([]byte, error)
    Unmarshal(data []byte, v interface{}) error
    Name() string
}
```

### 使用方式

```go
import (
    "github.com/go-kratos/kratos/v2/encoding"
    _ "github.com/go-kratos/kratos/v2/encoding/json"  // 注册 Codec
)

// 获取 Codec
jsonCodec := encoding.GetCodec("json")

// 序列化
bytes, _ := jsonCodec.Marshal(user)

// 反序列化
jsonCodec.Unmarshal(data, &user)
```

### 自定义 Codec

```go
import (
    "github.com/go-kratos/kratos/v2/encoding"
    "gopkg.in/yaml.v3"
)

const Name = "myyaml"

func init() {
    encoding.RegisterCodec(codec{})
}

type codec struct{}

func (codec) Marshal(v interface{}) ([]byte, error) {
    return yaml.Marshal(v)
}

func (codec) Unmarshal(data []byte, v interface{}) error {
    return yaml.Unmarshal(data, v)
}

func (codec) Name() string {
    return Name
}
```

## 监控指标

### 指标接口

```go
// Counter - 计数器
type Counter interface {
    With(lvs ...string) Counter
    Inc()
    Add(delta float64)
}

// Gauge - 状态指示器
type Gauge interface {
    With(lvs ...string) Gauge
    Set(value float64)
    Add(delta float64)
    Sub(delta float64)
}

// Observer - 观察指标（Histogram/Summary）
type Observer interface {
    With(lvs ...string) Observer
    Observe(float64)
}
```

### 使用示例

```go
import "github.com/go-kratos/kratos/v2/metrics"

// Counter - 统计请求数
counter := metrics.CounterWith("requests_total", "method", "GET")
counter.Inc()
counter.Add(1)

// Gauge - 统计当前连接数
gauge := metrics.GaugeWith("connections", "protocol", "grpc")
gauge.Set(100)
gauge.Add(10)
gauge.Sub(5)

// Observer - 统计请求延迟
observer := metrics.ObserverWith("request_duration_seconds", "path", "/api")
observer.Observe(0.125)
```

## 依赖注入 (Wire)

### Provider 定义

```go
// 提供配置文件
func NewConfig() *conf.Data { ... }

// 提供数据组件
func NewData(c *conf.Data) (*Data, error) { ... }

// 提供持久化组件
func NewUserRepo(d *data.Data) (*UserRepo, error) { ... }
```

### ProviderSet

```go
// internal/data/data.go
var ProviderSet = wire.NewSet(NewData, NewGreeterRepo)

// internal/biz/biz.go
var ProviderSet = wire.NewSet(NewGreeterUsecase)

// internal/server/server.go
var ProviderSet = wire.NewSet(NewGRPCServer, NewHTTPServer)

// internal/service/service.go
var ProviderSet = wire.NewSet(NewGreeterService)
```

### wire.go

```go
//go:build wireinject
// +build wireinject

package main

import (
    "helloworld/internal/biz"
    "helloworld/internal/data"
    "helloworld/internal/server"
    "github.com/go-kratos/kratos/v2"
    "github.com/google/wire"
)

func InitializeApp(c *conf.Bootstrap) (*kratos.App, func(), error) {
    panic(wire.Build(
        server.ProviderSet,
        data.ProviderSet,
        biz.ProviderSet,
        service.ProviderSet,
        newApp,
    ))
}
```

### 生成命令

```bash
cd cmd/helloworld
wire
```

## 最佳实践

### Makefile 模板

```makefile
.PHONY: init
init:
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
    go install github.com/go-kratos/kratos/cmd/kratos/v2@latest
    go install github.com/go-kratos/kratos/cmd/protoc-gen-go-http/v2@latest
    go install github.com/google/gnostic/cmd/protoc-gen-openapi@latest
    go install github.com/google/wire/cmd/wire@latest

.PHONY: api
api:
    protoc --proto_path=./api \
           --proto_path=./third_party \
           --go_out=paths=source_relative:./api \
           --go-grpc_out=paths=source_relative:./api \
           --go-http_out=paths=source_relative:./api \
           $(API_PROTO_FILES)

.PHONY: config
config:
    protoc --proto_path=./internal/conf \
           --proto_path=./third_party \
           --go_out=paths=source_relative:./internal/conf \
           $(CONF_PROTO_FILES)

.PHONY: generate
generate:
    go generate ./...
    go mod tidy

.PHONY: build
build:
    go build -o ./bin/ ./...

.PHONY: run
run:
    ./bin/helloworld -conf ./configs
```

### kratos-layout 初始化

```go
// cmd/server/main.go
package main

import (
    "context"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/go-kratos/kratos/v2"
    "github.com/go-kratos/kratos/v2/config"
    "github.com/go-kratos/kratos/v2/config/file"
    "github.com/go-kratos/kratos/v2/log"
    "github.com/go-kratos/kratos/v2/transport/grpc"
    "github.com/go-kratos/kratos/v2/transport/http"
)

func main() {
    // 加载配置
    c := config.New(
        config.WithSource(file.NewSource("configs")),
    )
    if err := c.Load(); err != nil {
        panic(err)
    }

    var bc conf.Bootstrap
    if err := c.Scan(&bc); err != nil {
        panic(err)
    }

    // 初始化 logger
    logger := log.With(log.NewStdLogger(os.Stdout),
        "ts", log.DefaultTimestamp,
        "caller", log.DefaultCaller,
        "service.id", id,
        "service.name", Name,
        "service.version", Version,
    )

    // 创建 App
    app, cleanup, err := wireApp(bc, logger)
    if err != nil {
        panic(err)
    }
    defer cleanup()

    // 启动
    if err := app.Run(); err != nil {
        panic(err)
    }
}
```

## 常见问题

### 1. Wire 生成失败

```bash
# 安装 wire
go install github.com/google/wire/cmd/wire@latest

# 生成
cd cmd/helloworld
wire
```

### 2. Proto 生成失败

```bash
# 检查 protoc 安装
protoc --version

# 检查 proto_path
kratos proto client api/helloworld/v1/hello.proto -p ./third_party
```

### 3. 项目创建失败

```bash
# 使用国内镜像
kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git
```

### 4. 配置热更新不生效

确保使用 `Watch` 方法监听配置变更，并正确处理回调。

### 5. 中间件顺序问题

中间件执行顺序：请求进入时按注册顺序，响应返回时按注册倒序。

## 参考资源

- [官方文档](https://go-kratos.dev/zh-cn/docs/)
- [GitHub 仓库](https://github.com/go-kratos/kratos)
- [项目模板](https://github.com/go-kratos/kratos-layout)
- [示例项目](https://github.com/go-kratos/examples)
- [错误规范](https://cloud.google.com/apis/design/errors)
- [Wire 文档](https://github.com/google/wire)
