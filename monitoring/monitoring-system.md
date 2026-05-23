# 系统监控体系

## 监控架构

```
┌─────────────────────────────────────┐
│          crontab（Linux 定时器）      │
├────────────────┬───────────────────┤
│ 5分钟一次       │ 15-30分钟一次       │
│ gateway-check   │ disk-check         │
│                 │ load-check         │
│                 │ network-check      │
├────────────────┴───────────────────┤
│           问题 → 自动修复            │
└─────────────────────────────────────┘
```

## Gateway 进程守护

`gateway-check.sh` 每 5 分钟检查一次，核心逻辑：

```bash
# 用端口查 PID（避免 pgrep 自匹配）
GATEWAY_PID=$(ss -tlnp "sport = :18789" | grep -oP 'pid=\K[0-9]+' | head -1)

if [ -z "$GATEWAY_PID" ]; then
    systemctl --user start openclaw-gateway   # 重启
elif curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:18789/ != 200; then
    kill "$GATEWAY_PID" && sleep 3            # 进程僵死 → 杀死等重启
    systemctl --user start openclaw-gateway
fi
```

### 防并发锁

```bash
LOCKFILE=/tmp/gateway-check.lock
exec 9>"$LOCKFILE"
flock -n 9 || exit 0   # 如果已有实例在跑，直接退出
```

## 磁盘监控

`disk-check.sh` 每 30 分钟检查：

```bash
USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$USAGE" -gt 90 ]; then
    # 发告警
fi
```

## 负载监控

`load-check.sh` 每 15 分钟检查：

```bash
LOAD=$(uptime | grep -oP 'load average: \K[0-9.]+')
CORES=$(nproc)
if [ "$(echo "$LOAD > $CORES * 0.8" | bc)" -eq 1 ]; then
    # 发告警
fi
```

## 网络监控

`network-check.sh` 每 15 分钟检查关键服务连通性：

```bash
for HOST in "8.8.8.8" "api.telegram.org" "api.github.com"; do
    ping -c 1 -W 3 "$HOST" >/dev/null || logger ...
done
```

## 定时任务一览

```bash
# 系统守护
*/5  * * * * gateway-check.sh     # Gateway 进程守护
*/15 * * * * load-check.sh        # 系统负载
*/15 * * * * network-check.sh     # 网络连通
*/30 * * * * disk-check.sh        # 磁盘空间

# 维护
0 6 * * * maintenance.js daily    # 每日维护
0 6 * * 1 maintenance.js weekly   # 每周维护
0 */4 * * * version-manager check  # 版本检查

# 其他
30 8 * * * weather-forecast.js    # 天气预报
0 3 * * 0 auto-discover.js        # 模型自动发现
0 0 * * * model-health-check.sh   # 模型健康
```