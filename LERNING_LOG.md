# OpenClaw 技术爬取 & 学习记录

## 数据来源

| 来源 | 仓库 | ⭐ Stars |
|------|------|---------|
| **官方主库** | openclaw/openclaw | **374k** |
| **汉化版** | MaoTouHU/OpenClawChinese | 350 |
| **文档站** | docs.openclaw.ai | — |

## 所学技术汇总

### 1. 官方 SKILL.md 元数据格式
```yaml
metadata:
  openclaw:
    emoji: "🐙"
    requires: { bins: ["gh"] }
    install:
      - { id: brew, kind: brew, formula: gh }
      - { id: apt, kind: apt, package: gh }
```

### 2. GitHub 官方最佳实践
- URL 直接操作: gh pr view https://github.com/owner/repo/pull/55
- API 缓存: gh api --cache 1h repos/owner/repo
- 重跑失败 job: gh run rerun <run-id> --failed
- GH_CONFIG_DIR 环境变量处理多账号

### 3. Healthcheck 审计方法论
- Context Inference (OS/权限/网络/备份)
- Read-only Checks (端口/防火墙/磁盘加密)
- Hardening Menu (3 种安全姿态)

### 4. 汉化版运维经验
- npm install -g @qingchencloud/openclaw-zh
- 一键安装脚本: curl -fsSL <url> | bash
- Docker 部署: docker pull 1186258278/openclaw-zh:latest
- 免费 AI 接口对接方案
- 设备配对 / token mismatch 解决方案

## 已反哺到仓库
- openclaw-agentics: skills 元数据格式升级
- openclaw-techniques: 新增官方参考文档
- openclaw-tools: healthcheck 审计方法论
