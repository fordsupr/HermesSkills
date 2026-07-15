# Telegram × Hermes 設定摘要

> 建立時間：2026-07-15  
> 狀態：✅ 已成功啟用

---

## 📋 設定歷程

### 1. 建立 Telegram Bot
- 使用 **@BotFather** 建立機器人 `hermes agent`
- 使用者名稱：**@Hermejasonli0918Bot**
- Bot Token：`8828553500:AAGr_***`（已更新）

### 2. 取得的 ID 資料

| 項目 | 值 | 說明 |
|------|-----|------|
| Bot ID | `8828553500` | 機器人的數字 ID（token 冒號前的數字） |
| 你的 User ID | `7544189719` | 你的個人 Telegram 帳號 ID |
| Bot Username | @Hermejasonli0918Bot | 機器人使用者名稱 |

### 3. 遭遇的問題與解決

| 問題 | 原因 | 解決方式 |
|------|------|----------|
| ❌ Gateway 無法連線 | Bot Token 中間有空格，token 損壞 | 從 @BotFather 重新產生新 Token |
| ❌ Chat not found | Home Channel 設為 `8828553500`（bot ID，不是有效聊天室） | 清空 Home Channel |
| ❌ 模型 provider 失敗 | gateway 使用本機 Qwen 模型（localhost:8080），且 base_url 指向 localhost | 切換至 DeepSeek V4 Flash，清除 base_url |
| ❌ 使用者被阻擋 | Allowed Users 設為 username 而非數字 User ID | 啟用 Allow All Users 開發模式 |

### 4. 最終設定

**Config (config.yaml):**
```yaml
model:
  default: deepseek-v4-flash
  provider: deepseek
  base_url: ""    # 使用 DeepSeek 預設 endpoint
```

**Env (.env):**
```env
TELEGRAM_BOT_TOKEN=8828553500:AAGr_***
TELEGRAM_ALLOW_ALL_USERS=true
TELEGRAM_ALLOWED_USERS=
TELEGRAM_HOME_CHANNEL=
```

### 5. 啟動方式

```bash
# 前景啟動
hermes gateway run --replace

# 背景服務（launchd）
hermes gateway start
hermes gateway stop
hermes gateway restart
hermes gateway status
```

---

## 🔒 建議後續安全性設定

等你確認機器人正常運作後，建議完成以下設定：

```bash
# 1. 限制只能你自己使用
# 編輯 .env，改用你的數字 User ID：
TELEGRAM_ALLOWED_USERS=7544189719

# 2. 關閉 Allow All Users
TELEGRAM_ALLOW_ALL_USERS=false

# 3. 設定 Home Channel（你的個人對話 ID 一樣是 7544189719）
TELEGRAM_HOME_CHANNEL=7544189719
```

---

## 📝 常用指令

| 指令 | 功能 |
|------|------|
| `/start` | 啟動對話 |
| `/help` | 查看所有指令 |
| `/model` | 查看或切換模型 |
| `/status` | 查看系統狀態 |
| `/platforms` | 查看平台連線狀態 |
| `/sethome` | 將當前對話設為 Home Channel |

---

*設定完成後，gateway 使用 DeepSeek V4 Flash 作為模型，回應時間約 5 秒，可正常對話。*
