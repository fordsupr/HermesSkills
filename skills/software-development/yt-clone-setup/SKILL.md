---
name: yt-clone-setup
description: "youtube-article-tool 一鍵部署 — git clone → uv 建 venv → 安裝 requirements.txt → 驗證 yt-dlp → 啟動伺服器。適用新機器或重灌後從零架起 YouTube 轉文章工具"
version: 1.0.0
author: Jason Li
platforms: [macos, linux]
metadata:
  hermes:
    tags: [uv, python, git-clone, setup, youtube-article-tool, fastapi]
---

# yt-clone-setup

> 從 git clone 到伺服器跑起來，全程用 uv 管理 Python 環境

---

## 一、前置檢查

依序確認，缺哪個就先裝哪個：

```bash
git --version        # 沒有 → xcode-select --install (macOS) 或 apt install git
uv --version         # 沒有 → curl -LsSf https://astral.sh/uv/install.sh | sh
```

安裝 uv 後需重開 shell 或 `source ~/.zshrc` 讓 PATH 生效。

---

## 二、Clone 專案

```bash
git clone <repo-url> youtube-article-tool
cd youtube-article-tool
```

若目錄已存在，改用更新流程：`cd youtube-article-tool && git pull`。

---

## 三、uv 建環境 + 安裝依賴

```bash
uv venv                                # 建 .venv
uv pip install -r requirements.txt    # 裝全部依賴（含 yt-dlp）
```

注意：
- 若出現 `Failed to hardlink files` 警告（cache 與專案在不同檔案系統），無害；要消掉可 `export UV_LINK_MODE=copy`。
- **不要**用系統 pip 或 conda 混裝，一律走 uv。

---

## 四、驗證

```bash
.venv/bin/yt-dlp --version        # 必須有輸出，伺服器用 subprocess 呼叫它
.venv/bin/python -c "import fastapi, uvicorn; print('ok')"
```

`yt-dlp` 是執行期的外部指令依賴：伺服器啟動時必須能在 PATH 找到它，
所以**啟動前一定要先 activate venv**（見下一節），否則會報
`[Errno 2] No such file or directory: 'yt-dlp'`。

---

## 五、設定 API Key（可選）

Gemini API key 兩種給法，擇一：

```bash
export GEMINI_API_KEY="你的key"    # 環境變數（建議）
```

或啟動後在網頁表單填入。**不要**把 key 寫進任何會 commit 的檔案；
專案 `.gitignore` 已含 `.env*`。

---

## 六、啟動

```bash
source .venv/bin/activate
./start.sh          # 或 uvicorn app.main:app --host 127.0.0.1 --port 8080 --reload
```

啟動前若 port 8080 被占用：

```bash
lsof -i :8080       # 列出占用 PID，由使用者決定 kill 或換 port
```

成功標誌：終端出現 `Uvicorn running on http://127.0.0.1:8080`，
瀏覽器開 http://127.0.0.1:8080 看到首頁。

---

## 七、常見問題

| 症狀 | 原因 | 解法 |
|------|------|------|
| `No such file or directory: 'yt-dlp'` | 沒 activate venv 就啟動 | `source .venv/bin/activate` 後重啟 |
| `請提供有效的 Gemini API Key` | 未設 GEMINI_API_KEY | 設環境變數或表單填入 |
| hardlink 警告 | cache 跨檔案系統 | `export UV_LINK_MODE=copy`（可忽略） |
| port 8080 占用 | 舊 server 還在跑 | `lsof -i :8080` 找 PID 再處理 |
