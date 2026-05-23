# 🎯 Skills 分支 — 自定义技能开发

本分支专注 OpenClaw 自定义技能开发。每个技能是一个独立的目录。

## 技能清单

| 技能 | 用途 | 目录 |
|------|------|------|
| gh-ops | GitHub 全操作 | local: `workspace/skills/gh-ops/` |
| github | GitHub 基础操作 | local: `workspace/skills/github/` |
| github-search | GitHub 仓库搜索 | local: `workspace/skills/github-search/` |

## 开发模板

```text
skill-name/
├── _meta.json     # 元数据（slug, version）
├── SKILL.md       # 技能说明（YAML frontmatter + 操作指南）
└── scripts/       # 可选辅助脚本
    └── xxx.mjs    # Node.js 脚本
```
