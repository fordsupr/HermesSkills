---
name: hermes-git-workflow
description: "透過 Hermes Agent 使用 Git 的完整工作流程 — 從初始化、日常 commit/push/pull、分支管理到救援操作，全部使用自然語言指令"
version: 1.0.0
author: Jason Li
platforms: [macos, linux, windows]
metadata:
  hermes:
    tags: [git, version-control, github, workflow, hermes-commands]
---

# Hermes Git Workflow

> 在 Hermes Agent 中使用自然語言操作 Git，從初始化到進階救援

---

## 一、前置檢查

當使用者說「幫我 git 同步」或「幫我 commit」時：

1. **檢查是否為 git repo**：`git status`（若非 repo，執行初始化流程）
2. **檢查 git 身份是否已設定**：`git config user.name` / `git config user.email`
3. **檢查 remote 是否存在**：`git remote -v`

---

## 二、初始化流程

當專案目錄尚未建立 git repo 時：

```bash
git init
git add -A
git commit -m "initial commit: <專案簡述>"
git branch -M main
```

### 建立遠端 repo 並推送

使用 `gh` CLI 建立 GitHub repo：

```bash
gh repo create <帳號>/<repo名稱> --private --description "<描述>" --push --source=.
```

或由使用者指定 remote URL 後：

```bash
git remote add origin <url>
git push -u origin main
```

---

## 三、日常 Git 操作對照表

使用者不一定會講出精確的 git 指令，需要從自然語言推斷：

### 3.1 查看狀態

| 使用者說 | 執行 |
|----------|------|
| 「幫我看 git 狀態」 | `git status` |
| 「這個目錄是 git repo 嗎？」 | `git status`（非 repo 時顯示致命錯誤） |
| 「最近改了什麼」 | `git status` + `git diff --stat` |
| 「幫我看 commit 歷史」 | `git log --oneline -10`（顯示最近 10 筆） |

### 3.2 提交變更

| 使用者說 | 執行 |
|----------|------|
| 「幫我 commit」 | `git add -A` → `git commit -m "<自動產生的訊息>"` → 顯示結果 |
| 「幫我 commit 訊息為 xxx」 | `git add -A` → `git commit -m "<使用者提供的訊息>"` |
| 「幫我 commit 並推送」 | `git add -A` → `git commit -m "..."` → `git push` |
| 「幫我 git 同步」 | `git pull` → `git add -A` → `git commit -m "..."` → `git push` |

### 3.3 Commit message 慣例

自動產生 commit message 時遵循 Conventional Commits：

| 類型 | 使用時機 | 範例 |
|------|----------|------|
| `feat:` | 新功能 | `feat: add user login API` |
| `fix:` | 修 bug | `fix: correct date format error` |
| `docs:` | 文件 | `docs: update README` |
| `refactor:` | 重構 | `refactor: extract validation logic` |
| `chore:` | 雜務 | `chore: update dependencies` |

### 3.4 推送與拉取

| 使用者說 | 執行 |
|----------|------|
| 「幫我推送到 GitHub」 | `git push` |
| 「幫我拉取最新」 | `git pull` |
| 「幫我同步遠端」 | `git fetch` + `git status`（顯示與遠端的差異） |

---

## 四、分支操作

| 使用者說 | 執行 |
|----------|------|
| 「幫我看有哪些分支」 | `git branch` |
| 「幫我開一個叫 xxx 的分支」 | `git checkout -b xxx` |
| 「幫我切到 main 分支」 | `git checkout main` |
| 「把 xxx 合併到 main」 | `git checkout main` → `git merge xxx` |
| 「刪除 xxx 分支」 | `git branch -d xxx`（已合併）或 `-D`（強制） |
| 「推分支到遠端」 | `git push -u origin <分支名>` |

---

## 五、救援操作

| 使用者說 | 執行 | 注意 |
|----------|------|------|
| 「後悔最後一次 commit」 | `git reset --soft HEAD~1` | 保留變更 |
| 「完全還原到上一個版本」 | `git reset --hard HEAD~1` | ⚠️ 破壞性，需確認 |
| 「改到一半想換分支」 | `git stash` → 切分支 → `git stash pop` | |
| 「推送被拒絕」 | `git pull` → 解決衝突 → `git add .` → `git commit -m "merge"` → `git push` | |
| 「取消 git init」 | `rm -rf .git` | ⚠️ 破壞性，需確認 |
| 「放棄對某個檔案的修改」 | `git checkout -- <檔案>` | |
| 「想改最後一次 commit 訊息」 | `git commit --amend -m "<新訊息>"` | 若已推送，需要 `--force` |

### 安全機制

執行下列破壞性操作前，**必須先請使用者確認**：

- `git reset --hard`
- `git push --force` / `git push --force-with-lease`
- `git branch -D <分支>`（強制刪除）
- `rm -rf .git`

> 範例：**⚠️ 即將執行：`git push --force origin main`，這會強制覆蓋遠端分支，確定要繼續嗎？**

---

## 六、首次使用引導

當使用者在一個非 git repo 目錄中說 git 相關指令時，自動引導初始化：

1. 檢查是否已安裝 git：`git --version`
2. 檢查 git 身份：`git config --global user.name`
3. 若尚未設定身份：提示設定名稱與 Email
4. 若尚未註冊 GitHub：引導至 `https://github.com/signup`
5. 詢問是否初始化 git repo

---

## 七、結合 OpenWiki 的同步流程

當使用者使用 OpenWiki 管理專案文件時：

```
openwiki/          ← code wiki（OpenWiki 自動產生）
wiki-personal/     ← 個人 wiki（symlink 到 ~/.openwiki/wiki/）
quant0050.md       ← 整合文件
```

對我說「幫我 git 同步」時：

```bash
git pull                          # 先拉取遠端最新
git add -A                        # 加入所有變更（含 wiki 更新）
git commit -m "sync: update wiki docs"  # 提交
git push                          # 推送
```

---

## 八、常用捷徑

以下斜線指令可在 Hermes session 中使用：

| 捷徑 | 等同於 |
|------|--------|
| `幫我 git 同步` | `git pull → add → commit → push` |
| `幫我 commit` | `git add -A → git commit` |
| `幫我推` | `git push` |
| `git 狀態` | `git status` |
| `最近改了什麼` | `git log --oneline -10` |

---

## 九、注意事項

1. **未註冊 Git 身份**：commit 會失敗並顯示 `Please tell me who you are`，需先 `git config --global user.name` 和 `user.email`
2. **沒有 remote**：push 會顯示 `No configured push destination`，需先 `git remote add origin <url>`
3. **外接硬碟 git**：在 `/Volumes/` 路徑下可能遇到 `GIT_DISCOVERY_ACROSS_FILESYSTEM` 限制，需先確認 git repo 存在
4. **DS_Store**：建議將 `.DS_Store` 加入 `.gitignore` 避免 macOS 系統檔案被追蹤
5. **大型檔案**：超過 100MB 的檔案無法 push 到 GitHub，需使用 Git LFS 或排除
