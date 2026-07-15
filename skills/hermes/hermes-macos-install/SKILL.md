---
name: hermes-macos-install
description: "在全新 macOS (Apple Silicon) 上完整安裝 Hermes Agent，含 DeepSeek provider、自訂 provider、Telegram gateway、外接卷宗配置，目錄結構與本機一致"
version: 1.0.0
author: Jason Li
platforms: [macos]
metadata:
  hermes:
    tags: [hermes, setup, installation, macos, deepseek, telegram, gateway]
---

# Hermes Agent — macOS 完整安裝指南

> 適用：macOS (Apple Silicon / arm64)，zsh
> 安裝後目錄結構與 `/Volumes/projectWrk/HermesAgent` 一致

---

## 目錄

1. [前置需求](#step-0--前置需求)
2. [決定安裝目錄](#step-1--決定安裝目錄hermes_home)
3. [執行官方 installer](#step-2--執行官方-installer)
4. [寫入 shell 環境](#step-3--寫入-shell-環境zshrc)
5. [首次設定 provider 與 API key](#step-4--首次設定-provider--api-key)
6. [自訂 provider（Ollama 本地模型）](#step-5--自訂-providerollama-本地模型)
7. [Telegram Gateway 設定](#step-6--telegram-gateway-設定)
8. [GitHub Skills 來源設定](#step-7--github-skills-來源設定)
9. [安裝 Hermes Skills](#step-8--安裝-hermes-skills)
10. [完整健檢](#step-9--完整健檢)
11. [更新 / 重裝 / 移除](#step-10--更新--重裝--移除)
12. [已知地雷](#已知地雷)

---

## Step 0 — 前置需求

macOS 內建 `curl` 即可；installer 會自動裝 `uv` 與受管 Python 3.11/3.12。不需要事先裝 Homebrew Python。

```bash
# 驗證
curl --version | head -1
```

### 選擇安裝路徑

本文件以外接卷宗 `/Volumes/projectWrk/HermesAgent` 為範例。若用本機磁碟，建議改為 `~/.hermes`。

> ⚠️ 外接卷宗未掛載時 hermes 會找不到家目錄——開機自啟或 cron 情境要先確認掛載。

---

## Step 1 — 決定安裝目錄（HERMES_HOME）

```bash
export HERMES_HOME=/Volumes/projectWrk/HermesAgent
mkdir -p "$HERMES_HOME"

# 驗證：目錄存在且可寫
touch "$HERMES_HOME/.write_test" && rm "$HERMES_HOME/.write_test" && echo OK
```

---

## Step 2 — 執行官方 installer

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

installer 會：
1. 安裝 `uv`
2. 下載 hermes-agent 原始碼到 `$HERMES_HOME/hermes-agent/`
3. 建立 Python 虛擬環境 `.venv`（Python 3.11）
4. 放置 `hermes` launcher 到 `~/.local/bin/`

```bash
# 驗證
ls "$HERMES_HOME/hermes-agent/.venv/bin/python" && ~/.local/bin/hermes --version
```

---

## Step 3 — 寫入 shell 環境（~/.zshrc）

```bash
grep -q HERMES_HOME ~/.zshrc || cat >> ~/.zshrc <<'EOF'

# Hermes Agent — ensure ~/.local/bin is on PATH
export HERMES_HOME=/Volumes/projectWrk/HermesAgent
export PATH="$HERMES_HOME/bin:$HOME/.local/bin:$PATH"
EOF
source ~/.zshrc

# 驗證
which hermes && echo "HERMES_HOME=$HERMES_HOME"
```

`$HERMES_HOME/bin/` 包含 `uv`/`uvx`（installer 自動放置），以及 `tirith`（安全掃描工具）。

---

## Step 4 — 首次設定 Provider & API Key

### 4.1 首次執行

```bash
hermes   # 首跑會進入 setup 精靈
```

精靈會引導設定 provider 與 API key。完成後金鑰與設定落在：

| 檔案 | 說明 |
|------|------|
| `$HERMES_HOME/config.yaml` | 主設定檔 |
| `$HERMES_HOME/auth.json` | OAuth tokens（權限 600） |
| `$HERMES_HOME/.env` | API keys（權限 600，勿入 git） |

```bash
# 驗證
hermes --version
ls -l "$HERMES_HOME/config.yaml" "$HERMES_HOME/auth.json"
```

### 4.2 DeepSeek Provider 設定

本機使用 **DeepSeek** 作為主要 provider：

```yaml
# $HERMES_HOME/config.yaml
model:
  default: deepseek-v4-flash
  provider: deepseek
  base_url: https://api.deepseek.com/v1
```

`.env` 需設定：
```bash
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

切換方式（session 中）：
```
/model deepseek-v4-flash
```

或變更 config：
```bash
hermes config set model.default deepseek-v4-flash
hermes config set model.provider deepseek
```

### 4.3 語言設定

```bash
hermes config set display.language zh-hant
```

---

## Step 5 — 自訂 Provider（Ollama 本地模型）

若需本地 GGUF 模型（如 Qwen3.5），透過 Ollama 以 OpenAI-compatible 方式註冊：

### 5.1 安裝 Ollama

```bash
brew install ollama
ollama serve   # 背景啟動
```

或下載 GGUF 並用 `llama-server` 啟動。

### 5.2 設定 custom_providers

在 `$HERMES_HOME/config.yaml` 新增 `custom_providers` 段落：

```yaml
custom_providers:
  - name: Ollama-Qwen3.5-9B
    base_url: http://127.0.0.1:11434/v1
    model: oamazonasgabriel/qwen3.5-9b:q4-16gbGPU
    models:
      oamazonasgabriel/qwen3.5-9b:q4-16gbGPU:
        context_length: 262144
```

### 5.3 切換至自訂 provider

在 session 中輸入：
```
/model Ollama-Qwen3.5-9B
```

> 若有多個自訂 provider，逐一加入 `custom_providers` 陣列即可。

---

## Step 6 — Telegram Gateway 設定

### 6.1 建立 Telegram Bot

1. 在 Telegram 中搜尋 [@BotFather](https://t.me/BotFather)
2. 輸入 `/newbot`，依指示建立 bot
3. 取得 bot token（格式：`1234567890:ABCdefGHIjklmNOPqrSTUvwxYZ`）

### 6.2 設定 Gateway Config

```bash
hermes gateway setup
# 選擇 telegram 平台，輸入 bot token
```

或直接編輯 `$HERMES_HOME/config.yaml`：

```yaml
gateway:
  platforms:
    telegram:
      home_channel_id: ''
      # bot_token 及 allowed_users 會寫入 .env
```

`.env` 需包含：
```bash
HERMES_TELEGRAM_BOT_TOKEN=1234567890:ABCdefGHIjklmNOPqrSTUvwxYZ
HERMES_TELEGRAM_ALLOWED_USERS=7544189719
HERMES_TELEGRAM_HOME_CHANNEL_ID=8828553500
```

> `ALLOWED_USERS` 必須是**數值 user ID**（非 @username），可透過 [@userinfobot](https://t.me/userinfobot) 查詢。

### 6.3 啟動 Gateway

```bash
hermes gateway run        # 前台（測試用）
hermes gateway install    # 安裝為背景服務
hermes gateway start      # 啟動服務
hermes gateway status     # 檢查狀態
```

---

## Step 7 — GitHub Skills 來源設定

本機使用 GitHub 上的 skills 倉庫來同步 skills 到多台機器。

```bash
# 加入 GitHub skill repo
hermes skills tap add https://github.com/fordsupr/HermesSkills.git

# 列出已安裝的 skills（確認連線正常）
hermes skills list
```

---

## Step 8 — 安裝 Hermes Skills

從 skills hub 安裝常用技能：

```bash
# macOS 相關
hermes skills install hermes-macos-install   # 本安裝指南

# 其他推薦技能
hermes skills install github-pr-workflow
hermes skills install github-code-review
hermes skills install test-driven-development
```

機構 skills（從 GitHub repo）：
```bash
hermes skills tap add https://github.com/fordsupr/HermesSkills.git
hermes skills install <skill-name>
```

---

## Step 9 — 完整健檢

```bash
echo "=== Hermes CLI ==="
hermes --version

echo "=== uv ==="
"$HERMES_HOME/bin/uv" --version

echo "=== Python (venv) ==="
"$HERMES_HOME/hermes-agent/.venv/bin/python" --version
# 必須是 3.11.x ~ 3.13.x（勿用 3.14）

echo "=== Config ==="
ls -la "$HERMES_HOME/config.yaml" "$HERMES_HOME/auth.json" "$HERMES_HOME/.env"

echo "=== Skills dir ==="
ls "$HERMES_HOME/skills/" | head -20

echo "=== Doctor ==="
hermes doctor

echo "=== Status ==="
hermes status
```

---

## Step 10 — 更新 / 重裝 / 移除

```bash
# 更新（設定與 state.db 會保留）
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash

# 完全移除
rm -rf "$HERMES_HOME" ~/.local/bin/hermes
# 並從 ~/.zshrc 刪掉 HERMES_HOME 兩行
```

---

## 安裝後目錄結構

```
$HERMES_HOME/
├── config.yaml            ← 主設定
├── auth.json              ← OAuth tokens
├── .env                   ← API keys
├── state.db               ← 會話資料庫
├── projects.db            ← 專案資料庫
├── hermes-agent/          ← 原始碼 + venv
│   ├── .venv/
│   └── ...
├── bin/
│   ├── uv                 ← Python 套件管理器
│   └── tirith             ← 安全掃描
├── skills/                ← 已安裝 skills
├── memories/              ← 持久記憶
├── sessions/              ← 會話記錄
├── cron/                  ← 排程任務
├── logs/                  ← 日誌
├── cache/                 ← 快取（web/browser 等）
├── gateway/               ← Gateway 狀態
├── audio_cache/           ← TTS 音檔快取
├── image_cache/           ← 圖片快取
├── plugins/               ← 外掛
├── platforms/             ← 平台適配器
├── hooks/                 ← 鉤子腳本
├── scripts/               ← 自訂腳本
├── agentWrk/              ← 工作目錄
├── kanban/                ← Kanban 任務板
├── sandboxes/             ← 沙箱
├── pets/                  ← 寵物系統
├── pairing/               ← 配對授權
├── shared/                ← 共享資料
└── 安裝步驟-macOS.md      ← 本安裝文件
```

---

## 已知地雷

### Python 版本
- **Python 3.14 不行**：pydantic-core 尚無 cp314 wheel，`requires-python <3.14` 會擋下，別硬設 `UV_PYTHON=3.14`。

### 環境變數汙染
- **PYTHONPATH 汙染**：從其他 Python 工具 session 裡跑 installer 會被自動 unset，屬正常行為。

### 外接硬碟
- **`/Volumes/projectWrk` 沒掛載時**所有 hermes 指令都會失敗，先 `ls /Volumes/projectWrk` 確認。

### Telegram Gateway
- **Bot token 不可有空格**：複製時容易貼進空白字元，導致認證失敗。
- **Allowed Users 必須是數值 ID**：`@username` 格式會被拒絕。

### Ollama / 本地模型
- 確保 `ollama serve` 或 `llama-server` 在 hermes 啟動前已執行。
- 自訂 provider 的 `base_url` 結尾建議加 `/v1`（如 `http://127.0.0.1:11434/v1`）。

---

## 速查表

| 用途 | 指令 |
|------|------|
| 啟動互動式 chat | `hermes` |
| 查 provider | `hermes model` |
| 切換 model | `/model <model-name>`（session 中） |
| 設定 API key | `hermes config set model.api_key <key>` |
| 編輯 config | `hermes config edit` |
| 健康檢查 | `hermes doctor` |
| 狀態總覽 | `hermes status` |
| 啟動 gateway | `hermes gateway start` |
| 安裝 skill | `hermes skills install <name>` |
| 瀏覽 skills | `hermes skills browse` |
| 加入 skills repo | `hermes skills tap add <url>` |
| 下載更新 | `hermes update` |
| 開新 session | `/new`（session 中） |
