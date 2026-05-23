# 配置管理最佳实践

## 备份策略

### 分层备份

| 级别 | 内容 | 频率 | 保留 |
|------|------|------|------|
| L0 auto | openclaw.json.auto-backup | Gateway 自动 | 1 个 |
| L1 manual | openclaw.json.manual-backup-* | 重大变更前 | 最近 3 个 |
| L2 full | backup/config-YYYYMMDD/ | 系统更新前 | 最近 2 个 |

### 完整备份清单

```bash
# 核心配置
~/.openclaw/openclaw.json      # 主配置（66KB）
~/.openclaw/gateway-owner.json # 网关持有者
~/.openclaw/exec-approvals.json # 执行审批

# 凭证
~/.openclaw/credentials/*.json  # Telegram 配对等

# 定时任务
~/.openclaw/cron/               # 内部 cron jobs
~/.openclaw/flows/              # 工作流注册
~/.openclaw/tasks/              # 任务运行记录

# 关键 workspace 文件
~/.openclaw/workspace/SOUL.md
~/.openclaw/workspace/USER.md
~/.openclaw/workspace/AGENTS.md
~/.openclaw/workspace/IDENTITY.md
~/.openclaw/workspace/TOOLS.md

# systemd 配置
~/.config/systemd/user/openclaw-gateway.service
~/.config/systemd/user/openclaw-gateway.service.d/*.conf

# crontab
crontab -l > crontab-backup.txt
```

### 备份命令

```bash
# 快速备份
DATE=$(date +%Y%m%d)
mkdir -p ~/.openclaw/backup/config-$DATE
cp ~/.openclaw/openclaw.json ~/.openclaw/backup/config-$DATE/
cp ~/.openclaw/gateway-owner.json ~/.openclaw/backup/config-$DATE/ 2>/dev/null
crontab -l > ~/.openclaw/backup/config-$DATE/crontab.txt
```

## 安全要点

### 严禁硬编码 Token

**不要** 在任何脚本中直接写 API token/bot token：

```javascript
// ❌ 错误
const TELEGRAM_BOT_TOKEN = '859236…';

// ✅ 正确 — 使用 openclaw CLI 发消息
execFileSync('openclaw', ['message', 'send', '--channel', 'telegram', ...]);
```

### Token 泄露应急

如果 bot token 或其他凭证暴露：

```bash
# 1. 立即在 Telegram BotFather 重新生成 token
# 2. 更新 openclaw.json 中的新 token
# 3. 删除含旧 token 的备份文件
find ~/.openclaw -name '*.bak' -exec grep -l 'token_here' {} \; -delete
```

### 文件权限

```bash
# 关键文件应为 600（仅所有者读写）
chmod 600 ~/.openclaw/openclaw.json
chmod 600 ~/.openclaw/credentials/*.json
chmod 700 ~/.openclaw/credentials/
```

## 恢复流程

```bash
# 从备份恢复
cp ~/.openclaw/backup/config-YYYYMMDD/openclaw.json ~/.openclaw/openclaw.json

# 恢复 systemd 配置
cp ~/.openclaw/backup/config-YYYYMMDD/stale-cleanup.conf \
   ~/.config/systemd/user/openclaw-gateway.service.d/
systemctl --user daemon-reload

# 恢复凭证
cp ~/.openclaw/backup/config-YYYYMMDD/credentials/*.json ~/.openclaw/credentials/

# 恢复 crontab
crontab ~/.openclaw/backup/config-YYYYMMDD/crontab.txt

# 重启 gateway
systemctl --user restart openclaw-gateway.service
```

## systemd 配置体系

```
~/.config/systemd/user/openclaw-gateway.service
├── [Unit]     Description、After、Wants、StartLimitBurst
├── [Service]  ExecStart、Restart、Environment
└── [Install]  WantedBy

~/.config/systemd/user/openclaw-gateway.service.d/
├── stale-cleanup.conf    # ExecStartPre + RestartSec
├── node-memory.conf      # NODE_OPTIONS=--max-old-space-size=1536
└── timeout.conf          # TimeoutStartSec=15, TimeoutStopSec=15
```

### 关键参数

```ini
Restart=always              # 崩溃自动重启
RestartPreventExitStatus=78 # Exit 78（端口占用）不自动重启
RestartSec=8                # 重启间隔
KillMode=control-group      # 杀整个进程组
```