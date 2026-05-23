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

## 📊 第二轮深度分析 (2026-05-23 11:18)

### 已扒取的 7 个高星项目全貌

| # | 项目 | ⭐ | 语言 | 核心功能 | 架构模式 |
|---|------|-----|------|---------|---------|
| 1 | **gogcli** | **7,516** | Go | Google Workspace CLI | 单体Go CLI，命令行映射到所有Google API |
| 2 | **mcporter** | 4,483 | Node | MCP Server CLI/Tool | CLI+MCP Server双模式，Auth/Config/Daemon/Codegen模块化 |
| 3 | **Peekaboo** | 4,439 | Swift+Node | macOS截图+VQA+自动化 | CLI命令 1:1 映射到 MCP tool，多AI Provider |
| 4 | **wacli** | 2,450 | Go | WhatsApp CLI | 安全优先设计（确认收件人+内容），守护进程同步 |
| 5 | **lobster** | 1,211 | TypeScript | 工作流引擎 | JSON管道/批准门/可恢复令牌/本地优先 |
| 6 | **imsg** | 1,124 | Go+Swift | iMessage CLI | 本地优先（直接读chat.db），filesystem事件流 |
| 7 | **openclaw-ansible** | 581 | Ansible | 部署自动化 | Tailscale+UFW+Fail2ban+Docker+node22+pnpm |

---

### 🧠 核心架构模式提取

#### 模式1: CLI ↔ MCP Server 双模式 (Peekaboo, mcporter)
```
CLI命令 → 1:1 → MCP Tools
```
Peekaboo 和 mcporter 都实现了"相同的工具集，两种入口"：
- CLI 模式：用户手动调用
- MCP 模式：AI Agent 通过 MCP 协议调用
- **本质优势**：一套代码，两种调用方式，减少维护成本

#### 模式2: JSON-first 结构化输出 (gogcli, lobster, imsg)
```
所有命令：
  - 标准输出: JSON 数据 (可机器解析)
  - 标准错误: 人类阅读的进度/提示
```
- gogcli: `--json` 和 `--plain` 输出区分
- lobster: "typed pipelines (objects/arrays), not text pipes"
- imsg: JSON schema 定义完整，`imsg completions llm` 生成 CLI 帮助供 AI 使用

#### 模式3: 安全设计模式 (wacli, gogcli, lobster)
- **wacli SKILL.md**: "Require explicit recipient + message text confirmation"
  - 在技能描述层声明安全约束
  - AI Agent 自然遵循这些约束
- **gogcli**: 内置命令 allowlist/denylist + read-only 审计模式
- **lobster**: 内置 approval gates（副作用操作自动暂停待审批）

#### 模式4: 去中心化/本地优先 (lobster, imsg)
- lobster: "Local-first execution. No new auth surface."
- imsg: 直接读取本地 chat.db，不需要网络往返
- **原则**: 能本地做的事不做远程调用

#### 模式5: 部署安全栈 (openclaw-ansible)
```
Tailscale VPN → UFW防火墙 → Fail2ban → unattended-upgrades → Docker隔离
```
形成完整的多层次安全防线。

---

### 🔬 具体代码学到的模式

#### Peekaboo CLI/MCP 映射
```
peekaboo image --mode screen  →  MCP Tool: screenshot
peekaboo see --app Safari     →  MCP Tool: see_accessibility
peekaboo click --on "button"   →  MCP Tool: click_element
peekaboo agent "natural lang" →  Agent Runtime
```

#### mcporter 模块化结构
```
mcporter list       →  列出MCP服务器
mcporter call       →  调用工具 (key=value / JSON / stdio)
mcporter auth       →  OAuth认证流程
mcporter config     →  配置管理
mcporter daemon     →  守护进程
mcporter generate-cli →  CLI代码生成
mcporter emit-ts    →  TypeScript类型生成
```

#### gogcli 企业级 CLI 设计
```
- 多账号支持（GOG_ACCOUNT 环境变量切换）
- 多认证方式（OAuth / ADC / Service Account）
- Docker 支持（持久化卷挂载）
- 安全剖面（allowlist/denylist + read-only）
- 自描述文档（docs/commands/README.md 自动生成）
```

#### lobster 管道基元
```
exec     — 运行 OS 命令
where    — 过滤
pick     — 选择字段
head     — 取前N条
json     — 渲染为 JSON
table    — 渲染为表格
approve  — 审批门（等待人工确认）
```
**核心**: 管道每一步都是 JSON typed，非文本流。

#### imsg 双栈架构
```
Layer 1: 公共 AppleScript API（发送/收件）
Layer 2: 高级 IMCore dylib 注入（编辑/撤回/群组管理/富文本/回执）
         → 需要 SIP 禁用，opt-in 模式
```

---

### 📚 对仓库的反哺建议

#### openclaw-tools 可新增的技能/工具
1. **lobster-workflow** — 引入 lobster 管道模式，把多步运维任务组合成流水线
   - `backup-pipeline`: 备份→验证→通知 三步流程
   - `deploy-pipeline`: 代码→测试→发布 审批流
2. **mcporter-mcp** — 包装为 MCP 工具，支持通过 mcporter 调用外部服务
3. **auto-deploy** — 吸收 openclaw-ansible 的安全部署模式

#### openclaw-agentics 可新增的技能
1. **security-first-skill** — 以 wacli 为模板的安全技能规范
2. **json-output-skill** — 以 gogcli 为蓝本的 JSON-first 输出规范
3. **cli-mcp-dual-skill** — CLI+MCP Server 双模式技能架构

#### openclaw-techniques 新增文档
1. `architecture/cli-mcp-dual-mode.md` — CLI↔MCP 双模式架构详解
2. `security/skill-security-design.md` — 技能安全约束设计模式
3. `patterns/json-first-output.md` — JSON-first 结构化输出模式

---

### 🎯 最终总结

| 学到的最重要 3 件事 | 来源 |
|-------------------|------|
| 1. CLI+MCP 双模式让同一个工具同时服务人和 AI | Peekaboo, mcporter |
| 2. 安全约束在 SKILL.md layer 用描述性语言即可生效 | wacli |
| 3. JSON typed pipeline 比文本管道更可靠 | lobster |

| 最想立即用上的 | 用途 |
|---------------|------|
| lobster 管道模式 | 把运维脚本组合成可审批可恢复的流水线 |
| gogcli 多账号模式 | GitHub 多 org 操作场景 |
| openclaw-ansible 安全栈 | 给 openclaw-web 加 Nginx + UFW 保护 |
