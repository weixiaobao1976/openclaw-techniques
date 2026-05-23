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

## 第二轮学习 (2026-05-23 11:14)

### 来源: 社区高分技能/扩展 (深度爬取)

| 项目 | ⭐ | 类型 | 学到 |
|------|----|------|------|
| **mcporter** | 4,483 | skills/ | MCP 服务器 CLI 工具格式（call/list/auth/daemon/codegen）|
| **Peekaboo** | 4,439 | extension | 截图 + VQA 视觉问答，独立 Go 扩展 |
| **wacli** | 2,450 | skills/ | WhatsApp CLI 安全规范：需要显式确认收件人+内容 |
| **gogcli** | 7,516 | extension | Google Workspace CLI（不在主分支，独立 repo）|
| **imsg** | 1,124 | extension | iMessage CLI 扩展 |
| **lobster** | 1,211 | extension | 工作流引擎 pipeline（独立 repo）|
| **openclaw-ansible** | 581 | 独立仓库 | 自动化部署：Tailscale VPN + UFW + Docker + Fail2ban + Node22 + pnpm |

### 🧠 关键学到的东西

#### 1. SKILL.md 的安全规范（从 wacli 学到）
```yaml
# 在技能描述中明确提出"需要显式确认"的安全约束
description: "Send WhatsApp via wacli. Require explicit recipient + message text confirmation."
```
wacli 的 SKILL.md 明确要求：**用户指定操作目标才执行**。这是安全设计的极佳参考。

#### 2. mcporter 的 6 大功能模块
```
Call     — server.tool key=value / JSON / stdio
Auth     — OAuth 认证流程
Config   — 配置管理（list/get/add/remove/import）
Daemon   — 后台守护进程
Codegen  — CLI 生成 + TypeScript 类型生成
```

#### 3. openclaw-ansible 的最佳实践模式
```yaml
- 隔离: Docker sandbox 隔离运行
- 网络: Tailscale VPN + UFW（仅SSH+Tailscale端口）
- 安全: Fail2ban + unattended-upgrades
- 部署: pnpm install -g openclaw@latest
- 用户: 专用系统用户 openclaw
```

#### 4. Peekaboo 的视觉问答能力
截图 + VQA（Visual Question Answering）模型，Agent 可自动截图并分析。

### 📚 反哺到我们仓库

- **openclaw-agentics/skills**: 将 mcporter 模块化架构模式应用到技能开发中
- **openclaw-tools**: 将 ansible 的自动化部署模式记录到 auto-repair 分支
- **openclaw-techniques**: 本学习日志更新到 learning 分支
