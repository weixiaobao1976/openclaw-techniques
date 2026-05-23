# gh-ops 自定义技能开发指南

## 技能概览

`gh-ops` 是基于 `gh` CLI 的 GitHub 全操作技能，覆盖仓库、Issues、PRs、CI/CD、Secrets、Releases、组织管理。

## 技能结构

```
~/.openclaw/workspace/skills/gh-ops/
├── _meta.json           # 元数据
├── SKILL.md             # 技能说明（完整操作指南）
└── scripts/
    ├── list-runs.mjs    # Workflow 运行状态查看
    └── changelog.mjs    # 基于 PR 的 CHANGELOG 生成
```

## _meta.json

```json
{
  "ownerId": "kn70pywhg0fyz996kpa8xj89s57yhv26",
  "slug": "gh-ops",
  "version": "1.0.0",
  "publishedAt": 1747993200000
}
```

## SKILL.md 模板

```markdown
---
name: gh-ops
description: "Full-stack GitHub ops via `gh` CLI: repos, issues, PRs, CI, releases, secrets, org management."
allowed-tools: [exec]
---
```

### 关键字段

| 字段 | 说明 |
|------|------|
| `name` | 技能名，用于被其他 agent 引用 |
| `description` | 触发描述，agent 根据它对技能做选择 |
| `allowed-tools` | 允许调用的工具，通常 `[exec]` |

## 脚本开发规范

### list-runs.mjs

```javascript
#!/usr/bin/env node
import { execSync } from 'child_process';

const [repo] = process.argv.slice(2).filter(a => !a.startsWith('--'));
const days = parseInt(process.argv[process.argv.indexOf('--days') + 1]) || 7;
const limit = parseInt(process.argv[process.argv.indexOf('--limit') + 1]) || 20;

const raw = execSync(`gh run list --repo ${repo} --limit ${limit} --json ...`, { encoding: 'utf-8', timeout: 15000 });
// 解析并格式化输出...
```

### 注意事项

1. 使用 `#!/usr/bin/env node` shebang
2. 使用 `execSync/execFileSync`，避免 spawn 的异步问题
3. 使用 `gh` CLI 而非直接调 GitHub REST API（自动使用已认证 token）
4. 设置合理的 timeout（通常 15-30s）
5. 用 `--repo owner/repo` 明确指定仓库

## 其他技能参考

仓库中还有以下技能可供参考：

```
~/.openclaw/workspace/skills/
├── github/                  # 基础 GitHub 操作（gh CLI）
├── github-search/           # GitHub 仓库搜索（REST API）
│   └── scripts/
│       ├── github-search.mjs
│       └── repo-detail.mjs
├── github-trending-cn/      # GitHub 趋势中文分析
└── github-actions-generator/ # GitHub Actions 工作流生成
```

## 验证技能语法

```bash
# 验证 YAML frontmatter
python3 -c "
import yaml
with open('SKILL.md') as f:
    fm = f.read().split('---',2)[1]
    d = yaml.safe_load(fm)
print(f'name: {d[\"name\"]}, tools: {d.get(\"allowed-tools\")}')
"

# 验证 JS 脚本语法
node --check scripts/xxx.mjs
```