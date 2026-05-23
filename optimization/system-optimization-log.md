# 系统优化全记录

## 概况

| 轮次 | 优化项 | 效果 |
|------|--------|------|
| v1 | 旧备份清理、日志轮转、gateway-check 修复 | -54MB 磁盘 |
| v2 | symlink 清理、内存上限、启动超时、TCP keepalive | 启动加速 + 内存管控 |

---

## v1：磁盘与稳定性优化

### 清理内容

| 项目 | 清理前 | 清理后 |
|------|--------|--------|
| `daily.log` | **25 MB**（每晚报错） | **4 KB**（修复PATH后不再爆增） |
| `fs-safe-replace` 临时文件 | 22 个 | 0 |
| openclaw.json 旧备份 | 22 个 | 12 |
| backup 目录 | 4 个 | 2 |
| weather 日志 JSON | 238 行 | 30 行 |

### 修复 crontab PATH

**问题：** maintenance.js 中调 `openclaw health --json`，但 crontab 环境的 PATH 不包含 `~/.npm-global/bin`，报 `openclaw: not found`，导致每天 25MB 日志暴涨。

**修复：** 在 crontab 首行注入完整 PATH

```bash
PATH=/usr/local/bin:/usr/bin:/home/zbc2/.local/bin:/home/zbc2/.npm-global/bin:/home/zbc2/bin:/bin
```

### gateway-check 自匹配修复

**问题：** `pgrep -f "node.*openclaw.*gateway"` 会匹配 crontab 执行的 shell 脚本路径（含 `gateway`）

**修复：** 改为 `ss -tlnp "sport = :18789"` 端口精确查 PID，加防并发锁

## v2：性能与启动优化

### symlink 清理

**问题：** workspace/skills 中有 **29 个 symlink** 指向外部目录（`~/.agents/skills/` 和 `~/.claude/skills/`），每次 agent session 启动扫描这些 symlink 耗时 **6.4 秒**，加载后又被跳过。

**分类：**
- 19 个 `gstack-*` 指向 Claude/skills（OpenClaw 跳过的死链接）
- 10 个指向 `.agents/skills`（与本地技能重复）

**处理：** 全部移除（备份到 `backup/symlinks-backup-*`）

### Node.js 内存上限

**问题：** Gateway RSS 440MB，V8 heap 仅 4MB，差异在于 code cache + native addon

**修复：** 加 `--max-old-space-size=1536`

```ini
[Service]
Environment=NODE_OPTIONS=--max-old-space-size=1536
```

### 缩短 systemd 超时

| 参数 | 旧值 | 新值 | 原因 |
|------|------|------|------|
| TimeoutStartSec | 30s | 15s | 实际启动仅需 ~7s |
| TimeoutStopSec | 30s | 15s | 正常关闭更快 |

### TCP keepalive 调优

```bash
sysctl -w net.ipv4.tcp_keepalive_time=300
sysctl -w net.ipv4.tcp_keepalive_intvl=10
sysctl -w net.ipv4.tcp_keepalive_probes=5
```

从默认 2 小时降为 5 分钟，加快断连检测。

## 日志轮转

### gateway 日志（每天 23:55）

```bash
55 23 * * * /usr/bin/logrotate -s /tmp/openclaw/logrotate.state /tmp/openclaw-logrotate.conf
```

配置：
```
/tmp/openclaw/openclaw-*.log {
    daily        rotate 3
    maxsize 50M  compress
    delaycompress  missingok  notifempty  copytruncate
}
```

### weather 日志（每天 00:10）

```bash
10 0 * * * tail -200 ~/.openclaw/logs/weather-forecast.log > tmp && mv tmp weather-forecast.log
```

## 最终系统状态

| 指标 | 优化前 | 优化后 |
|------|--------|--------|
| ~/.openclaw/ 磁盘 | 209 MB | 155 MB |
| Gateway 内存 | 440 MB | 364 MB |
| skills symlink | 29 个 | 0 |
| crontab 任务 | 12 个 | 14 个（含轮转） |
| 系统维护日志 | 25MB | 4KB |
| 启动延迟 | 6.4s（扫描 symlink） | 即刻加载 |