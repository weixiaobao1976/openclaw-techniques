# Monitoring 分支

本分支专注 Gateway 监控和各通道健康检查。

## 监控组件

- `gateway-check.sh` — 端口级进程守护（ss + curl 双验证）
- `disk-check.sh` — 磁盘空间告警（>90% 预警）
- `load-check.sh` — 系统负载告警
- `network-check.sh` — 网络连通性（Telegram / GitHub / Google）
- `pre-network-check.sh` — 启动前网络就绪检查（/usr/local/bin/openclaw-check-network.sh）

## 告警阈值

| 指标 | 警告 | 严重 |
|------|------|------|
| CPU 负载 | > 核数×0.8 | > 核数 |
| 内存使用 | > 80% | > 90% |
| 磁盘 | > 80% | > 90% |
| Gateway 响应 | > 3s | 不可达 |
