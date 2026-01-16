# Go Kratos Skill

这是一个专为Kratos微服务框架设计的技能包，提供完整的微服务开发指南、最佳实践和项目模板。

## 技能内容

### 核心功能

- **快速入门指南** - 从零开始构建Kratos微服务
- **完整架构说明** - DDD分层架构和依赖注入
- **Proto定义** - gRPC服务定义和错误定义
- **业务逻辑** - Service、Biz、Data层完整实现
- **配置管理** - 统一配置管理和热更新
- **中间件开发** - 内置和自定义中间件
- **服务注册与发现** - Etcd、Consul等注册中心集成
- **错误处理** - 统一错误定义和处理
- **日志系统** - 结构化日志和链路追踪
- **监控指标** - Prometheus集成和性能监控
- **测试指南** - 单元测试和集成测试
- **部署运维** - Docker和Kubernetes部署

### 项目模板

包含完整的微服务项目模板，包括：

- `cmd/helloworld/main.go` - 应用入口文件
- `configs/config.yaml` - 完整配置文件
- `go.mod` - 依赖管理文件

## 使用方法

### 1. 安装技能

将技能包导入到Claude Code中，技能会自动在相关场景下触发。

### 2. 使用场景

当用户需要以下帮助时，此技能会自动激活：

- 创建新的Kratos微服务
- 开发gRPC/HTTP API
- 配置服务注册与发现
- 实现DDD分层架构
- 配置链路追踪和监控
- 编写微服务测试
- 部署微服务应用

### 3. 项目模板使用

技能包含完整的微服务项目模板，可以直接使用：

```bash
# 使用kratos工具创建项目
kratos new helloworld

# 或者使用技能中的项目模板文件
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

## 依赖工具

技能中提到的相关工具：

- [Kratos](https://github.com/go-kratos/kratos) - 微服务框架
- [Protobuf](https://developers.google.com/protocol-buffers) - 协议定义
- [Wire](https://github.com/google/wire) - 依赖注入
- [OpenTelemetry](https://opentelemetry.io/) - 链路追踪
- [Prometheus](https://prometheus.io/) - 监控指标
- [Etcd](https://etcd.io/) - 服务注册中心

## 贡献

欢迎提交问题和改进建议！

## 许可证

MIT License