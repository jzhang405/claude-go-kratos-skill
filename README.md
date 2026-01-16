# Go Kratos Skill

专为 Kratos 微服务框架设计的 Claude AI 技能包,提供完整的微服务开发指南、最佳实践和项目模板。

## 关于 Kratos

[Kratos](https://go-kratos.dev/) 是一套轻量级 Go 微服务框架,包含大量微服务相关框架及工具。名字来源于希腊神话中的战神奎托斯。

### 设计原则

- **简单**: 不过度设计,代码平实简单
- **通用**: 通用业务开发所需要的基础库功能
- **高效**: 提高业务迭代的效率
- **稳定**: 基础库可测试性高,覆盖率高,有线上实践安全可靠
- **高性能**: 性能高,但不特定为了性能做 hack 优化
- **扩展性**: 良好的接口设计,支持扩展实现
- **容错性**: 为失败设计,鲁棒性高
- **工具链**: 包含大量工具链

### 核心特性

- **APIs**: 协议通信以 HTTP/gRPC 为基础,通过 Protobuf 定义
- **Errors**: 通过 Protobuf 的 Enum 作为错误码定义
- **Metadata**: 在协议通信 HTTP/gRPC 中,通过 Middleware 规范化服务元信息传递
- **Config**: 支持多数据源方式,进行配置合并铺平,支持动态配置
- **Logger**: 标准日志接口,可方便集成三方 log 库
- **Metrics**: 统一指标接口,默认集成 Prometheus
- **Tracing**: 遵循 OpenTelemetry 规范定义,实现微服务链路追踪
- **Encoding**: 支持 Accept 和 Content-Type 进行自动选择内容编码
- **Transport**: 通用的 HTTP/gRPC 传输层,实现统一的 Middleware 插件支持
- **Registry**: 实现统一注册中心接口,可插件化对接各种注册中心

## 技能内容

### 核心功能

- **快速入门指南** - 从零开始构建 Kratos 微服务
- **完整架构说明** - DDD 分层架构和依赖注入
- **Proto 定义** - gRPC 服务定义和错误定义
- **业务逻辑** - Service、Biz、Data 层完整实现
- **配置管理** - 统一配置管理和热更新
- **中间件开发** - 内置和自定义中间件
- **服务注册与发现** - Etcd、Consul 等注册中心集成
- **错误处理** - 统一错误定义和处理
- **日志系统** - 结构化日志和链路追踪
- **监控指标** - Prometheus 集成和性能监控
- **测试指南** - 单元测试和集成测试
- **部署运维** - Docker 和 Kubernetes 部署

### 项目模板

包含完整的微服务项目模板,包括:

- `cmd/helloworld/main.go` - 应用入口文件
- `configs/config.yaml` - 完整配置文件
- `go.mod` - 依赖管理文件

### 标准目录结构 (kratos-layout)

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
├── internal/                      # 内部代码(不对外暴露)
│   ├── biz/                       # 业务逻辑层(DDD Domain)
│   │   ├── README.md
│   │   ├── biz.go
│   │   └── greeter.go
│   ├── conf/                      # 配置结构定义(proto 生成)
│   │   ├── conf.pb.go
│   │   └── conf.proto
│   ├── data/                      # 数据访问层(DDD Repo 实现)
│   │   ├── README.md
│   │   ├── data.go
│   │   └── greeter.go
│   ├── server/                    # Server 层(HTTP/gRPC 实例)
│   │   ├── grpc.go
│   │   ├── http.go
│   │   └── server.go
│   └── service/                   # Service 层(DDD Application)
│       ├── README.md
│       ├── greeter.go
│       └── service.go
├── third_party/                   # 第三方 proto 依赖
└── go.mod
```

#### 各层职责

| 层级 | 职责 | 说明 |
|------|------|------|
| **api** | API 定义 | proto 文件及生成代码 |
| **cmd** | 入口 | 应用启动入口,Wire 注入点 |
| **internal/biz** | 业务逻辑 | 领域模型、业务规则、Repo 接口定义 |
| **internal/data** | 数据访问 | 实现 biz 定义的 Repo 接口 |
| **internal/service** | 服务层 | DTO 到 DO 转换,协调 biz 层 |
| **internal/server** | 传输层 | HTTP/gRPC Server 创建配置 |

## 使用方法

### 1. 安装 Kratos CLI 工具

```bash
# 安装 CLI 工具
go install github.com/go-kratos/kratos/cmd/kratos/v2@latest

# 验证安装
kratos -v

# 升级到最新版本
kratos upgrade
```

### 2. 环境准备

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

### 3. 创建并运行项目

```bash
# 1. 创建项目(使用默认模板)
kratos new helloworld

# 国内使用 Gitee 镜像
kratos new helloworld -r https://gitee.com/go-kratos/kratos-layout.git

# 2. 进入项目目录
cd helloworld

# 3. 安装依赖
go mod download
go get github.com/google/wire/cmd/wire@latest

# 4. 生成代码
go generate ./...

# 5. 运行
kratos run
```

### 4. 测试接口

```bash
# HTTP API
curl 'http://127.0.0.1:8000/helloworld/kratos'
# {"message":"Hello kratos"}

# 错误响应
curl 'http://127.0.0.1:8000/helloworld/error'
# {"code":404,"reason":"USER_NOT_FOUND","message":"user not found: error"}
```

### 5. 使用技能

将技能包导入到 Claude Code 中,技能会在相关场景下自动触发,帮助您:

- 创建新的 Kratos 微服务
- 开发 gRPC/HTTP API
- 配置服务注册与发现
- 实现 DDD 分层架构
- 配置链路追踪和监控
- 编写微服务测试
- 部署微服务应用

## 常用命令

### 项目创建

```bash
kratos new <project-name>              # 创建新项目
kratos new helloworld -b main           # 指定分支
kratos new helloworld -r <repo-url>    # 使用自定义模板
```

### Proto 操作

```bash
kratos proto add <path>                # 添加 proto 模板
kratos proto client <path>             # 生成客户端代码
kratos proto server <path>             # 生成服务端代码
```

### 运行

```bash
kratos run                             # 运行项目
kratos changelog                       # 查看更新日志
```

### Makefile 示例

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

## 技能特色

### 完整覆盖

- 从基础服务创建到高级微服务功能
- 包含实际生产环境的最佳实践
- 提供完整的错误处理和监控方案

### 实用性强

- 所有代码示例都经过测试
- 包含实际微服务中的常见问题解决方案
- 提供性能优化和安全考虑

### 易于使用

- 清晰的代码示例和说明
- 渐进式的学习路径
- 完整的项目模板

## 参考资源

### 官方资源

- [Kratos 官方文档](https://go-kratos.dev/zh-cn/docs/)
- [GitHub 仓库](https://github.com/go-kratos/kratos)
- [项目模板](https://github.com/go-kratos/kratos-layout)
- [示例项目](https://github.com/go-kratos/examples)

### 社区

- [Wechat Group](https://github.com/go-kratos/kratos/issues/682)
- [Discord Group](https://discord.gg/BWzJsUJ)
- QQ Group: 716486124

### 依赖工具

- [Kratos](https://github.com/go-kratos/kratos) - 微服务框架
- [Protobuf](https://developers.google.com/protocol-buffers) - 协议定义
- [Wire](https://github.com/google/wire) - 依赖注入
- [OpenTelemetry](https://opentelemetry.io/) - 链路追踪
- [Prometheus](https://prometheus.io/) - 监控指标
- [Etcd](https://etcd.io/) - 服务注册中心

## 开源协议

Kratos is MIT licensed. See the [LICENSE](https://github.com/go-kratos/kratos/blob/main/LICENSE) file for details.

## 贡献

欢迎提交问题和改进建议!

---

**Made with ❤️ for Go Kratos community**
