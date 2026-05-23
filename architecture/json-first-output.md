# JSON-first 结构化输出模式

> 来自 gogcli (7.5k⭐) + lobster (1.2k⭐) + imsg (1.1k⭐) 的最佳实践

## 核心原则

```
stdout = JSON data (机器解析)
stderr = 人类提示 (可选)
```

## 对比传统 Shell 管道

```bash
# ❌ 传统方式（基于文本解析）
df -h | grep /dev/sda1 | awk '{print $5}' | tr -d '%'

# ✅ JSON-first 方式（结构化数据）
json-report.mjs --json | jq '.system.disk.usagePct'
```

## gogcli 的输出模式

| 模式 | stdout | stderr | 用途 |
|------|--------|--------|------|
| `--json` | 完整 JSON | 无 | AI/脚本消费 |
| `--plain` | 摘要文本 | JSON | 人类阅读+记录 |
| 默认 | JSON | 进度提示 | 双模式 |

## lobster 的管道基元

```
exec     → 执行（输入JSON，输出JSON）
where    → 过滤（类似 SQL WHERE）
pick     → 选择字段
head     → 取前N条
json     → JSON 渲染
table    → 表格渲染
approve  → 审批门
```

### 示例
```bash
lobster 'exec --json "echo [1,2,3]" | where "0>=0" | json'
```

## imsg 的 JSON Schema 设计

- 完整的 JSON schema 定义每个命令的输出结构
- `imsg completions llm` 为 AI 生成 CLI 用法的上下文帮助
- CLI → JSON 双向解析

## 对仓库的实践

### openclaw-tools/json-report.mjs（已实现）
```javascript
const format = process.argv.includes('--pretty') ? 'pretty' 
  : process.argv.includes('--plain') ? 'plain' : 'json';
// stdout → JSON 数据，stderr → 人类提示
```

### 建议
1. 所有 shell 脚本改为 JSON-first 输出
2. 使用 `--json` 作为默认模式
3. 用 `--plain` 做人类友好摘要
4. 错误信息走 stderr，数据走 stdout
