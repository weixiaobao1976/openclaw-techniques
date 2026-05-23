# 自动化运维配置

## crontab PATH 修复

### 问题

maintenance.js 中调用 `openclaw health --json`，但 crontab 默认 PATH 不包含 `~/.npm-global/bin`，导致：

```
/bin/sh: 1: openclaw: not found
```

**后果：** 每晚 maintenance 备份失败，25MB 日志暴涨。

### 修复

在 crontab 首行设置完整 PATH：

```bash
PATH=/usr/local/bin:/usr/bin:/home/zbc2/.local/bin:/home/zbc2/.npm-global/bin:/home/zbc2/bin:/bin
```

### 当前 crontab

```bash
# ── 维护 ──
0 6 * * *     maintenance.js daily          # 每日维护
0 6 * * 1     maintenance.js weekly         # 每周维护
0 */4 * * *   version-manager.js check      # 版本检查
@reboot       version-manager.js check      # 启动时检查

# ── 模型 ──
0 3 * * 0     auto-discover.js              # 模型自动发现
0 0 * * *     model-health-check.sh         # 模型健康检查

# ── 工具 ──
30 8 * * *    weather-forecast.js            # 天气预报
0 3 * * *     cleanup-videos.sh              # 清理视频项目

# ── 监控（系统级守护） ──
*/5 * * * *   gateway-check.sh               # Gateway 进程守护
*/15 * * * *  load-check.sh                  # 系统负载
*/15 * * * *  network-check.sh               # 网络连通性
*/30 * * * *  disk-check.sh                  # 磁盘空间

# ── 日志轮转 ──
55 23 * * *   logrotate ...                  # 网关日志轮转
10 0 * * *    weather-rotator.sh             # 天气日志截断
```

## OpenClaw 内部 Cron 任务

在 `~/.openclaw/cron/jobs.json` 中还有 OpenClaw 内部定时任务：

```json
{
  "alert-analyzer": {
    "schedule": "0 8 * * *",
    "message": "请分析系统监控告警"
  },
  "system-health-report": {
    "schedule": "0 9 * * 1",
    "message": "生成周度系统健康报告"
  },
  "model-list-refresh": {
    "schedule": "0 3 * * 0",
    "message": "检查是否有新的可用模型"
  }
}
```

## 日志轮转配置

### Gateway 日志

位置：`/tmp/openclaw-logrotate.conf`

```conf
/tmp/openclaw/openclaw-*.log {
    daily
    rotate 3
    maxsize 50M
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
    nocreate
}
```

每天 23:55 执行：`logrotate -s /tmp/openclaw/logrotate.state /tmp/openclaw-logrotate.conf`

### Weather 日志

每天 00:10 截断为最后 200 行。

## 验证自动化

```bash
# 检查 crontab 是否加载
crontab -l

# 检查 logrotate 能否正常执行
/usr/bin/logrotate -d /tmp/openclaw-logrotate.conf

# 检查 weather 日志大小
wc -l ~/.openclaw/logs/weather-forecast.log
```