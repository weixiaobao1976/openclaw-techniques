# CLI ↔ MCP Server 双模式架构

> 来自 Peekaboo (4.4k⭐) + mcporter (4.5k⭐) 的最佳实践分析

## 核心原则

**同一套工具，两个入口** — CLI 给人用，MCP 给 AI Agent 用。

## 实现模式

```
┌─────────────────┐     ┌──────────────────┐
│   CLI 用户       │     │  AI Agent (Claude)│
│   peekaboo image │     │  MCP 协议调用      │
└────────┬────────┘     └────────┬─────────┘
         │                       │
         ▼                       ▼
    ┌─────────────────────────────────┐
    │    同一套核心逻辑 (Core Lib)      │
    │  - screenshot                   │
    │  - click_element                │
    │  - see_accessibility            │
    └─────────────────────────────────┘
```

## 优点

| 维度 | 传统分开写 | 双模式 |
|------|-----------|--------|
| 维护成本 | 两套代码 | 一套核心 |
| 一致性 | 容易 drift | 始终一致 |
| 测试 | 两套测试 | 一套核心测试 |
| 文档 | 两套文档 | 自动生成 |

## Peekaboo 的例子

```bash
# CLI 模式
peekaboo image --mode screen --retina --path ~/Desktop/screen.png

# MCP 模式（同样操作，协议驱动）
# MCP 请求: { "tool": "screenshot", "args": { "mode": "screen", "retina": true } }
```

## mcporter 的例子

```bash
# CLI 模式
mcporter call linear.list_issues team=ENG limit:5

# MCP 模式
# 同一个核心，MCP server 包装
npx -y @steipete/peekaboo  # 启动 MCP server
```

## 对我们仓库的启发

- `openclaw-tools` 里的运维脚本可以同时提供 CLI + JSON API 入口
- `openclaw-agentics` 里的技能可以同时设计给人看和给 AI 用的格式
- `openclaw-web` 可以作为 CLI 操作的可视化替代
