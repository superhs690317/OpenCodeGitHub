# OpenCode 連線 GitHub 實作紀錄

## 文件資訊

- 專案名稱：`OpenCodeGitHub`
- 執行日期：2026-06-02
- 執行環境：Windows PowerShell / OpenCode CLI
- 參考文件：`02-連接-GitHub.md`

---

## 目標

依照 `02-連接-GitHub.md` 的內容，逐步完成以下事項：

1. 檢查本機 Git 與 GitHub CLI。
2. 驗證 Git 使用者資訊。
3. 確認 GitHub CLI 登入狀態。
4. 設定 OpenCode 的 GitHub MCP。
5. 建立或重用測試 repo，實際推送測試頁。
6. 啟用 GitHub Pages 並確認站台網址。

---

## 執行前狀態

- 工作目錄：`C:\Users\super\Documents\OpenCode\GitHub`
- 環境：OpenCode CLI（非 Codex Desktop）
- GitHub 帳號中已存在公開測試 repo：`superhs690317/github-test`

---

## 步驟與結果

### 步驟 0：環境檢查

執行結果如下：

- `git --version`
  - 結果：`git version 2.53.0.windows.2`
- `gh --version`
  - 結果：`gh version 2.89.0 (2026-03-26)`
- `git config --global user.name`
  - 結果：`Handsome`
- `git config --global user.email`
  - 結果：`superhs690317@gmail.com`

判讀：

- Git 與 GitHub CLI 均可正常使用
- Git 使用者資訊完整
- `gh` 版本 2.89+，可用 `gh auth token` 取得憑證設定 GitHub MCP

### 步驟 1：確認 GitHub CLI 登入狀態

- `gh auth status`
  - 帳號：`superhs690317`
  - Active account：`true`
  - Git operations protocol：`https`
  - Token scopes：`gist`, `read:org`, `repo`, `workflow`

判讀：

- GitHub CLI 已登入，無需重新授權
- 權限包含 `repo`，可支援建立、推送、修改 repo

### 步驟 2：設定環境變數 `GITHUB_TOKEN`

GitHub MCP Server 需要一組有效的 GitHub 憑證。從已登入的 `gh` CLI 取得 token 並設為環境變數：

```powershell
$env:GITHUB_TOKEN = gh auth token
```

驗證 token 已正確設定：

```powershell
$env:GITHUB_TOKEN
```

### 步驟 3：設定 OpenCode GitHub MCP

在原 `opencode.json` 中已存在 NotebookLM MCP，本次加入 GitHub MCP（遠端伺服器模式）：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "notebooklm": {
      "type": "local",
      "command": ["notebooklm-mcp"],
      "enabled": true,
      "timeout": 300000
    },
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer {env:GITHUB_TOKEN}"
      },
      "oauth": false,
      "enabled": true
    }
  },
  "experimental": {
    "mcp_timeout": 300000
  }
}
```

> 註：此 GitHub MCP Server 是 GitHub 官方提供的遠端 API 服務，不需在本機安裝任何額外套件。

### 步驟 4：驗證 GitHub MCP 連線

在 OpenCode 對話中驗證 MCP 是否正常運作：

- OpenCode 可透過 MCP 讀取 GitHub repo 資訊
- 可列出已安裝的帳號與可存取 repo
- 實測可見 repo 包含既有測試 repo：`superhs690317/github-test`

判讀：

- GitHub MCP 已成功連接
- OpenCode 可直接在對話中查詢 repo、issue、PR

### 步驟 5：重用既有測試 repo

因 GitHub 上已存在 `superhs690317/github-test`，本次沿用既有 repo 進行驗證。

檢查遠端 repo 狀態：

- `gh repo view superhs690317/github-test --json name,visibility,url`
- 結果：
  - repo 名稱：`github-test`
  - 可見性：`PUBLIC`
  - URL：`https://github.com/superhs690317/github-test`

### 步驟 6：建立測試頁並提交

建立 `index.html`：

```html
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8">
  <title>OpenCode GitHub 連接成功</title>
</head>
<body>
  <h1>Hello！OpenCode GitHub 連接成功！</h1>
</body>
</html>
```

提交過程：

- `git add index.html`
- `git commit -m "建立 OpenCode GitHub 連線測試頁"`

### 步驟 7：推送到 GitHub

執行：

- `git push origin master`

結果：

- push 成功

### 步驟 8：確認 GitHub Pages

查詢 Pages 狀態：

- `gh api repos/superhs690317/github-test/pages`
- 結果：
  - 站台已啟用
  - 網址：`https://superhs690317.github.io/github-test/`
  - 來源分支：`master`
  - HTTPS：啟用

---

## 成功驗證項目

| 項目 | 狀態 |
|------|------|
| Git 版本檢查 | ✅ |
| GitHub CLI 安裝與登入 | ✅ |
| Git 使用者資訊設定 | ✅ |
| OpenCode GitHub MCP 設定 | ✅ |
| GitHub MCP 對話內查詢 | ✅ |
| 本機 commit 與 push | ✅ |
| GitHub Pages 連線確認 | ✅ |

---

## 本次踩坑與處理方式

### 1. MCP 設定方式差異

- 現象：
  - Codex Desktop 時期使用 GUI 的 GitHub connector 進行授權。
  - OpenCode CLI 無 GUI 介面，需改為 MCP 設定。
- 處理：
  - 透過 `opencode.json` 的 `mcp.github` 區塊設定為遠端 MCP Server。

### 2. `gh mcp` 指令不存在（v2.89.0）

- 現象：
  - 查遍官方文件後發現 `gh` CLI 本身沒有內建 `mcp` 子指令。
  - 即使 `gh --version` 為 2.89.0，`gh mcp` 仍回報 unknown command。
- 處理：
  - 改用 GitHub 官方提供的遠端 MCP Server：`https://api.githubcopilot.com/mcp/`。
  - 需搭配 `GITHUB_TOKEN` 環境變數（取自 `gh auth token`）進行驗證。

### 3. `GITHUB_TOKEN` 環境變數需手動設定

- 現象：
  - OpenCode 啟動時若無 `GITHUB_TOKEN`，GitHub MCP Server 會回應 401。
- 處理：
  - 在每次啟動 OpenCode 前執行 `$env:GITHUB_TOKEN = gh auth token`。
  - 建議將該指令寫入 `$PROFILE` 以自動載入。

---

## 最終驗證（2026-06-02）

在完成所有設定後，執行 7 項全面測試確認系統狀態：

| 測試項目 | 結果 |
|----------|------|
| 1. 工具檢查 — git / gh CLI 可用 | ✅ |
| 2. GitHub CLI 登入狀態 — superhs690317, repo scope | ✅ |
| 3. Git 使用者資訊 — Handsome / superhs690317@gmail.com | ✅ |
| 4. `GITHUB_TOKEN` 環境變數 — 已設定（40 字元） | ✅ |
| 5. GitHub Pages — `github-test` 已 built | ✅ |
| 6. `opencode.json` — 根目錄與子專案格式皆正確 | ✅ |
| 7. 檔案完整性 — 6 個檔案內容皆符合預期 | ✅ |

---

## 最終成果

| 項目 | 網址 |
|------|------|
| 測試 repo | `https://github.com/superhs690317/github-test` |
| GitHub Pages | `https://superhs690317.github.io/github-test/` |
| 本機專案 | `C:\Users\super\Documents\OpenCode\GitHub` |

---

## 建議後續用法

完成這次設定後，後續可直接對 OpenCode 下列指令：

- 幫我把這個專案推到 GitHub
- 幫我建立新的 GitHub repo
- 幫我把這個靜態網頁開成 GitHub Pages
- 幫我檢查目前 repo 狀態並同步遠端

---

## 附註

- 本次實作同時保留既有 Codex 時期建立的 `github-test` repo，未另外新建。
- OpenCode CLI 的工作流與 Codex Desktop 不同，差異主要體現在 MCP 設定方式與 connector 管理。
- 本專案已上傳至 GitHub repo：`superhs690317/OpenCodeGitHub`
