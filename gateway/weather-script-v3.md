# 天气预报脚本 v3 改造

## 背景

原脚本使用 `https.get` 请求 wttr.in，无重试无超时，遇到 `ECONNRESET` 直接崩溃。
且硬编码了 Telegram bot token，存在安全风险。

## 改造要点

### 1. 增加重试机制

```javascript
function fetchWithRetry(url, retries = 3) {
  return new Promise((resolve, reject) => {
    const attempt = (n) => {
      const req = https.get(url, { timeout: 15000 }, (res) => {
        let data = '';
        res.on('data', c => data += c);
        res.on('end', () => resolve(data));
      });
      req.on('error', (err) => {
        if (n > 1) setTimeout(() => attempt(n - 1), 5000);
        else reject(err);
      });
    };
    attempt(retries);
  });
}
```

### 2. 移除硬编码 token

**改前：** 脚本中直接写 `const TELEGRAM_BOT_TOKEN = '859236…'`

**改后：** 使用 `openclaw message send --channel telegram --target "178274859" --message "..."`

### 3. 改用 execFileSync 安全传递参数

```javascript
execFileSync('openclaw', [
  'message', 'send',
  '--channel', 'telegram',
  '--target', '178274859',
  '--message', msg
], { timeout: 15000 });
```

避免 shell 转义问题。

### 4. 精简日志输出

新版不发 Telegram API 响应到日志，避免日志暴涨。

## 部署

脚本路径：`~/.openclaw/scripts/weather-forecast.js`

crontab：`30 8 * * * /usr/bin/node ~/.openclaw/scripts/weather-forecast.js >> ~/.openclaw/logs/weather-forecast.log`