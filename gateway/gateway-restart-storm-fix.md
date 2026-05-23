# Gateway 重启风暴修复

## 背景

Gateway 重启时反复断连，00:51~00:54 八分钟内重启了 8 次。

## 根因分析

### 问题1：ExecStopPost 自杀（元凶）

`stale-cleanup.conf` 中：

```bash
ExecStopPost=-/bin/bash -c 'for pid in $(pgrep -f "openclaw.*gateway"); do kill -9 "$pid"; done'
```

**机制：** systemd 重启时 `ExecStopPost` 执行，`pgrep -f "openclaw.*gateway"` 匹配到**新启动的 gateway 进程**，将其杀死 → systemd 检测到进程被异常杀死 → 自动重启 → 又杀死新进程 → **无限循环**。

### 问题2：ExecStartPre 误杀

```bash
ExecStartPre=-/bin/bash -c 'for pid in $(pgrep -f "openclaw.*gateway"); do kill -9 "$pid"; done'
```

同样使用 `pgrep -f` 模糊匹配，可能误杀系统上其他含 "gateway" 路径的进程。

### 问题3：间隔过短

`RestartSec=5` 只有 5 秒，端口释放不充分导致 `EADDRINUSE`。

---

## 修复方案

### 修改 stale-cleanup.conf

```ini
[Service]
# 仅通过端口号精确查杀旧 gateway 进程
ExecStartPre=-/bin/bash -c 'for pid in $(ss -tlnp "sport = :18789" | grep -oP "pid=\K[0-9]+"); do kill "$pid"; sleep 1; done; exit 0'

# 给端口释放时间
ExecStartPre=/bin/sleep 3

# 移除 ExecStopPost（元凶）
# 让 systemd 管理进程生命周期

# 加大重启间隔，防止端口冲突
RestartSec=8
```

### 关键变更

| 项目 | 旧配置 | 新配置 |
|------|--------|--------|
| 杀进程方式 | `pgrep -f` 模糊匹配 | `ss -tlnp` 精确端口查 PID |
| ExecStopPost | 暴力杀进程 ← 元凶 | **已移除** |
| 重启间隔 | 5 秒 | 8 秒 |

### 验证

```bash
# 确认配置生效
systemctl --user daemon-reload
systemctl --user show openclaw-gateway.service --property=ExecStartPre,ExecStopPost,RestartSec

# 重启测试
systemctl --user restart openclaw-gateway.service

# 观察启动日志
journalctl --user -u openclaw-gateway.service -f
```

## 恢复步骤

如果配置导致问题：

```bash
# 恢复备份
cp ~/.openclaw/backup/config-20260523/stale-cleanup.conf \
   ~/.config/systemd/user/openclaw-gateway.service.d/stale-cleanup.conf
systemctl --user daemon-reload
```

---

## 附录：完整 systemd 配置

目錄：

```bash
~/.config/systemd/user/openclaw-gateway.service.d/
├── stale-cleanup.conf    # 端口精确杀进程 + 防重启风暴
├── node-memory.conf      # Node.js 内存上限 1.5GB
└── timeout.conf          # 超时缩短至 15s
```

相关文件：

- 主服务文件: `~/.config/systemd/user/openclaw-gateway.service`
- 网络检查预脚本: `/usr/local/bin/openclaw-check-network.sh`