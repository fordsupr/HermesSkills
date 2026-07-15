---
name: telegram-gateway-setup
description: "完整的 Telegram Gateway 設定流程：建立 Bot、取得 Token、設定 Hermes 整合、排除常見問題"
version: 1.0.0
author: Hermes Agent
platforms: [macos, linux, windows]
metadata:
  hermes:
    tags: [telegram, gateway, messaging, setup]
---

# Telegram Gateway 設定流程

## 前置需求

- 一個 Telegram 帳號
- Hermes Gateway 已安裝（`hermes gateway status` 可檢查）

## 步驟

### 1. 建立 Telegram Bot

1. 打開 Telegram，搜尋 **@BotFather**
2. 發送 `/newbot`，依指示設定名稱和使用者名稱
3. 複製取得的 **Bot Token**（格式：`123456:ABC-def_ghi`）

### 2. 取得你的 User ID

1. 搜尋 **@userinfobot**
2. 發送 `/start`
3. 記錄它回覆的數字 ID（例如 `7544189719`）

### 3. 設定 Hermes .env

編輯 Hermes 的 `.env` 檔案（`hermes config env-path` 可查路徑）：

```env
# Telegram Bot Token
TELEGRAM_BOT_TOKEN=你的_bot_token

# 安全性：只允許特定使用者（設為數字 User ID，不是 @username）
TELEGRAM_ALLOWED_USERS=你的_user_id

# 開發模式：允許所有人使用（測試用，上線前關閉）
TELEGRAM_ALLOW_ALL_USERS=false

# Home Channel：通知與 cron 送達的預設對話
TELEGRAM_HOME_CHANNEL=你的_user_id

# Home Channel 顯示名稱
TELEGRAM_HOME_CHANNEL_NAME=Hermes Agent
```

### 4. 設定 Gateway 模型

確認 config.yaml 使用可用的雲端模型（避免本機模型未啟動導致錯誤）：

```bash
hermes config set model.provider deepseek
hermes config set model.default deepseek-v4-flash
hermes config set model.base_url ""
```

### 5. 啟動 Gateway

```bash
# 啟動背景服務
hermes gateway start

# 或前景測試
hermes gateway run --replace

# 檢查狀態
hermes gateway status
```

### 6. 驗證

1. 在 Telegram 中搜尋你的 Bot 使用者名稱
2. 發送 `/start`
3. 正常應收到 Hermes 回覆

## 常見問題

### ❌ Chat not found
- **原因**：Home Channel ID 設為 Bot ID 或不存在的聊天室
- **解決**：清空 Home Channel 或設為你的 User ID

### ❌ InvalidToken / token rejected
- **原因**：Bot Token 損壞（含空格）或過期
- **解決**：去 @BotFather 重新產生 `/newbot` 或 `/token`

### ❌ Connection error / model provider failed
- **原因**：gateway 使用本機模型但服務未啟動，或 base_url 指向錯誤 endpoint
- **解決**：切換至雲端 provider 並清除自訂 base_url

### ❌ Blocked unauthorized user
- **原因**：Allowed Users 設為 @username 而非數字 ID
- **解決**：從 @userinfobot 取得數字 User ID，或暫時啟用 Allow All Users

## 注意事項

- Bot Token 是敏感資訊，不要公開分享
- 正式使用前建議關閉 `TELEGRAM_ALLOW_ALL_USERS`
- Gateway logs 位於 `~/hermes/logs/gateway.log` 和 `~/hermes/logs/errors.log`
- 修改 config/.env 後需要重啟 gateway 才能生效

---

## 在其他機台安裝此 Skill

此 skill 已發布於 GitHub：

```
https://github.com/fordsupr/HermesSkills.git
```

### 方式 1：直接 URL 安裝（最簡單）

```bash
hermes skills install \
  https://raw.githubusercontent.com/fordsupr/HermesSkills/main/skills/hermes/telegram-gateway-setup/SKILL.md
```

### 方式 2：加入為 Tap 來源（方便日後更新）

```bash
# 加入 GitHub repo 作為 skill 來源
hermes skills tap add https://github.com/fordsupr/HermesSkills.git

# 安裝 skill
hermes skills install telegram-gateway-setup
```

### 更新 Skill

當來源 repo 有更新時，其他機台只需執行：

```bash
hermes skills update
```

即可自動同步最新版本。
