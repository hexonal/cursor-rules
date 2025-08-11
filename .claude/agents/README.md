# Claude System Agents for Go Development

## Available Agents

系统已注册以下 Go 开发专业 Agents：

| Agent | 触发词 | 功能描述 |
|-------|--------|----------|
| `go-developer` | @developer, @dev | Go 代码开发 |
| `go-reviewer` | @reviewer, @review | 代码审查 |
| `go-tester` | @tester, @test | 测试编写 |
| `go-architect` | @architect, @arch | 架构设计 |
| `go-performance` | @performance, @perf | 性能优化 |
| `go-concurrency` | @concurrency, @concurrent | 并发编程 |
| `go-security` | @security, @sec | 安全审查 |
| `go-error` | @error, @err | 错误处理 |
| `go-techdoc` | @techdoc, @doc | 技术文档 |

## 使用方法

### 1. 直接调用
在对话中使用触发词激活特定 agent：
```
使用 @developer 帮我实现一个 REST API
使用 @reviewer 审查这段代码
使用 @tester 编写测试用例
```

### 2. Agent 格式说明
每个 agent 文件都使用 YAML front matter 格式：
```yaml
---
name: agent-name
description: Agent description
model: sonnet
---

Agent content in Markdown
```

## Agent 功能概览

### 测试相关
- **go-tester**: 编写单元测试、集成测试、基准测试

### 开发相关
- **go-developer**: 代码开发、API 设计、重构
- **go-architect**: 系统架构、微服务设计、技术选型
- **go-concurrency**: 并发编程、goroutine 管理、channel 模式

### 质量保证
- **go-reviewer**: 代码审查、质量检查、最佳实践
- **go-security**: 安全审计、漏洞检测、安全加固
- **go-error**: 错误处理设计、调试支持

### 优化与文档
- **go-performance**: 性能分析、内存优化、瓶颈识别
- **go-techdoc**: 技术方案文档、架构图绘制

## 注意事项

1. Agents 使用 Markdown 格式，包含 YAML front matter
2. `model: sonnet` 指定使用的 AI 模型
3. 所有 agents 都集成了 MCP 工具支持
4. Agents 会根据上下文自动选择合适的工具

## 更新历史

- 2025-08-11: 创建系统级 agents
- 所有 agents 使用标准 YAML front matter 格式