# OpenCode 懶人包 #02：連接 GitHub

> 版本：v1.1（OpenCode CLI 版）
> 更新日期：2026-06-02

> 本懶人包可獨立執行：會先檢查 Git、GitHub CLI、登入狀態，再用網頁端登入完成 GitHub CLI 授權，接著設定 OpenCode 的 GitHub MCP 整合。

---

## 這個懶人包會幫你做什麼？

讓 OpenCode CLI 可以幫你操作 GitHub，包括：

- 建立 GitHub repo
- 把 OpenCode 做好的網頁、教材、互動 HTML 推送上線
- 開啟 GitHub Pages，讓學生掃 QR Code 就能開啟
- 之後用自然語言完成 commit、push、Pages 發布
- 在 OpenCode 對話中讀取 GitHub repo、issue、PR 內容

> [!important]
> GitHub 連線不是只靠 OpenCode 本身完成，而是靠電腦上的 Git、GitHub CLI，以及 GitHub 帳號授權。OpenCode 的角色是幫你檢查、執行、判讀錯誤、一步一步帶你完成。

> [!tip]
> 完整連線分成兩段：`gh` CLI 負責本機 push / 建 repo / 開 Pages；OpenCode 的 MCP 設定負責讓 OpenCode 在對話裡讀取 GitHub repo、issue、PR。

---

## 適用情境

本懶人包以 **OpenCode CLI（Windows）** 為主線，並使用 PowerShell 指令。

也可套用到：

| 工具 | 說明 |
|------|------|
| OpenCode CLI | 推薦，新手照本懶人包走 |
| OpenCode VS Code 擴充 | 可沿用同樣的 Git / gh 設定 |

---

## 先備條件

- [ ] OpenCode CLI 已安裝並能正常執行
- [ ] 有 GitHub 帳號；如果沒有，請先到 <https://github.com/signup> 註冊
- [ ] 電腦有網路連線
- [ ] 願意在瀏覽器完成一次 GitHub 授權

---

## 請 OpenCode 幫我執行以下步驟

> [!warning]
> 以下是給 OpenCode 讀的操作流程。遇到需要登入、授權、輸入驗證碼的地方，OpenCode 會停下來請使用者手動完成。

---

## 步驟零：環境檢查

請 OpenCode 先檢查目前環境，不要直接假設工具都已安裝。

### 0.1 檢查 Git

```powershell
git --version
```

成功時會看到類似：

```text
git version 2.53.0.windows.2
```

如果沒有安裝 Git：

```powershell
winget install --id Git.Git --accept-source-agreements --accept-package-agreements
```

安裝完若 OpenCode 還是找不到 `git`，請重新啟動終端機。

### 0.2 檢查 GitHub CLI

```powershell
gh --version
```

如果沒有安裝 GitHub CLI：

```powershell
winget install --id GitHub.cli --accept-source-agreements --accept-package-agreements
```

也可以到 GitHub CLI 官方頁面下載安裝：

<https://cli.github.com/>

### 0.3 檢查 Git 使用者資訊

```powershell
git config --global user.name
git config --global user.email
```

如果沒有設定，請使用者提供姓名與 email，再設定：

```powershell
git config --global user.name "你的姓名"
git config --global user.email "你的email@example.com"
```

---

## 步驟一：確認 GitHub 登入狀態

先檢查是否已登入：

```powershell
gh auth status
```

成功時應該看到類似：

```text
github.com
  ✓ Logged in to github.com account your-github-name
  - Active account: true
  - Git operations protocol: https
  - Token scopes: 'gist', 'read:org', 'repo'
```

重點確認：

| 欄位 | 要看到什麼 |
|------|------------|
| account | 是你的 GitHub 帳號 |
| Active account | `true` |
| Git operations protocol | `https` |
| Token scopes | 至少要包含 `repo` |

---

## 步驟二：用網頁端登入 GitHub

如果 `gh auth status` 顯示尚未登入，請執行：

```powershell
gh auth login --web --git-protocol https
```

接著會進入網頁端授權流程：

1. PowerShell 會顯示一組一次性驗證碼。
2. 瀏覽器通常會自動開啟 GitHub 授權頁面。
3. 如果瀏覽器沒有自動開啟，手動開啟：

```text
https://github.com/login/device
```

4. 在 GitHub 頁面輸入驗證碼。
5. 點選授權。
6. 回到 OpenCode / PowerShell，確認登入完成。

登入後再檢查一次：

```powershell
gh auth status
```

---

## 步驟三：設定 OpenCode GitHub MCP

OpenCode 沒有內建 GUI connector，而是透過 MCP（Model Context Protocol）伺服器來整合 GitHub。OpenCode 使用 GitHub 官方提供的遠端 MCP Server（`api.githubcopilot.com/mcp/`），不需在本機安裝額外套件。

### 3.1 設定環境變數 `GITHUB_TOKEN`

GitHub MCP Server 需要一組 GitHub 憑證（Personal Access Token）來驗證身份。最簡單的方式是從已登入的 `gh` CLI 取得 token：

```powershell
$env:GITHUB_TOKEN = gh auth token
```

如果要讓每次開啟 PowerShell 時自動設定，請將上面這行加到你的 PowerShell 設定檔：

```powershell
notepad $PROFILE
```

在檔案末端加入：

```powershell
$env:GITHUB_TOKEN = gh auth token
```

### 3.2 設定 opencode.json

在專案根目錄（或 `~/.config/opencode/`）的 `opencode.json` 中加入 GitHub MCP 設定：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "github": {
      "type": "remote",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer {env:GITHUB_TOKEN}"
      },
      "oauth": false,
      "enabled": true
    }
  }
}
```

> [!tip]
> `{env:GITHUB_TOKEN}` 是 OpenCode 的變數語法，會自動讀取環境變數 `GITHUB_TOKEN`，不需要把 token 寫死在設定檔中。

### 3.3 驗證 GitHub MCP 是否成功

設定完成後，在 OpenCode 對話中輸入：

```text
幫我確認 GitHub MCP 是否已連接，並列出我可以存取的 repo。
```

成功時，OpenCode 應該能讀取 GitHub 上的 repo 資訊。

### 3.4 MCP 設定失敗時怎麼辦

如果 OpenCode 回報無法連接 GitHub MCP：

1. 確認 `GITHUB_TOKEN` 環境變數已正確設定：`$env:GITHUB_TOKEN`
2. 確認 `gh auth status` 顯示已登入（token 才有效）
3. 重新啟動 OpenCode CLI
4. 若仍失敗，用除錯指令確認：`opencode mcp debug github`

---

## 步驟四：建立測試 repo 驗證

為了確認 OpenCode 真的能寫入 GitHub，建議建立一個測試 repo。

### 4.1 建立測試資料夾

```powershell
New-Item -ItemType Directory -Path "$env:USERPROFILE\Documents\github-test" -Force
Set-Location "$env:USERPROFILE\Documents\github-test"
```

### 4.2 建立測試網頁

```powershell
@"
<!doctype html>
<html lang="zh-Hant">
<head>
  <meta charset="utf-8">
  <title>GitHub 連接成功</title>
</head>
<body>
  <h1>Hello！GitHub 連接成功！</h1>
</body>
</html>
"@ | Set-Content -Encoding UTF8 -Path ".\index.html"
```

### 4.3 初始化 Git 並送上 GitHub

```powershell
git init
git add .
git commit -m "建立 GitHub 連線測試頁"
gh repo create github-test --public --source=. --push
```

### 4.4 開啟 GitHub Pages

```powershell
gh api repos/{owner}/github-test/pages -X POST -f "source[branch]=master" -f "source[path]=/"
```

如果上面指令失敗，可能是 repo 還沒完全建立完成。等 30 秒後再試一次，或請 OpenCode 幫你到 GitHub 網頁檢查 Pages 設定。

GitHub Pages 網址通常是：

```text
https://你的帳號.github.io/github-test/
```

請使用者打開網址確認是否看到：

```text
Hello！GitHub 連接成功！
```

> [!warning]
> GitHub Pages 第一次部署可能需要 1 到 3 分鐘。看到 404 時，先等一下再重新整理，不一定是失敗。

---

## 步驟五：保留或刪除測試 repo

測試完成後，請 OpenCode 詢問使用者：

```text
測試成功了，這個 github-test repo 要保留，還是刪除？
```

如果要刪除 GitHub 上的測試 repo：

```powershell
gh repo delete github-test --yes
```

如果要刪除本機測試資料夾：

```powershell
Remove-Item -LiteralPath "$env:USERPROFILE\Documents\github-test" -Recurse -Force
```

---

## 完成後可以怎麼用？

| 你說的話 | OpenCode 會做的事 |
|----------|------------------|
| 「幫我把這個專案推到 GitHub」 | 檢查 diff、commit、push |
| 「幫我建立 GitHub repo」 | 建 repo、設定 remote、推送 |
| 「幫我把這個網頁上線」 | 推送 repo、設定 GitHub Pages、回報網址 |
| 「幫我同步 GitHub」 | 檢查目前 repo 狀態，需要時 commit / push |

---

## OpenCode 環境下的注意事項

### 1. OpenCode 可能需要授權才能讀寫 GitHub CLI 設定

在 OpenCode CLI 裡，有時直接執行 `gh --version` 或 `gh auth status` 會遇到類似：

```text
failed to read configuration: open C:\Users\<你>\AppData\Roaming\GitHub CLI\config.yml: Access is denied.
```

這通常不是 GitHub CLI 壞掉，而是 OpenCode 需要你允許它讀取 GitHub CLI 的設定位置。

處理方式：

1. 讓 OpenCode 重新執行檢查，並同意授權。
2. 授權後再跑：

```powershell
gh auth status
```

3. 看到登入帳號與 `repo` scope 後再繼續。

### 2. GitHub MCP Server 與 GitHub CLI 是兩件事

| 類型 | 用途 |
|------|------|
| GitHub MCP Server（遠端 API） | 讓 OpenCode 在對話中查 repo、issue、PR 等 GitHub 資料 |
| GitHub CLI (`gh`) | 讓 OpenCode 在你的電腦上建立 repo、commit、push、開 Pages |

如果只是要讓 OpenCode 在本機專案裡幫你 commit / push，`gh` CLI 是最直接的路線。

如果要在對話中直接讀 GitHub 內容、查 PR、看 issue，則要設定 GitHub MCP。完整新手流程建議兩個都設定。

### 3. 不要把 token 寫進 AGENTS.md

AGENTS.md 可以記錄：

```md
## GitHub

GitHub 帳號：your-github-name
預設 repo：private
需要發布網頁時使用 GitHub Pages
```

不要記錄：

```md
GitHub token: ghp_xxxxx
```

token、密碼、一次性驗證碼都不要寫進 repo 或對外筆記。

---

## 踩坑筆記

| 狀況 | 原因 | 解法 |
|------|------|------|
| `gh --version` 顯示 Access is denied | OpenCode 被擋在 GitHub CLI 設定檔外 | 允許 OpenCode 讀取後重跑 |
| `gh auth status` 沒有登入 | 尚未完成 GitHub 授權 | 執行 `gh auth login --web --git-protocol https` |
| GitHub MCP 無法連接 | `GITHUB_TOKEN` 未設定或已失效 | 執行 `$env:GITHUB_TOKEN = gh auth token` 後重啟 OpenCode |
| MCP 看得到帳號但找不到 repo | OpenCode 尚未有 GitHub 專案 | 先在 GitHub 上建立 repo |
| 瀏覽器沒有自動開 | 裝置登入頁沒被自動喚起 | 手動開 `https://github.com/login/device` |
| push 被拒絕 | 權限不足或不是正確帳號 | 重跑 `gh auth status`，確認帳號與 `repo` scope |
| GitHub Pages 404 | Pages 尚未部署完成 | 等 1 到 3 分鐘再重整 |
| OpenCode 可以讀 repo 但不能 push | MCP 可讀不等於本機 git 有權限 | 檢查 `gh auth status` 與 git remote |

---

## 更新紀錄

| 日期 | 版本 | 更新內容 |
|------|------|----------|
| 2026-06-02 | v1.0 | OpenCode CLI 初版：移除 Codex Desktop 專屬 connector 流程，改為 GitHub MCP Server |
| 2026-06-02 | v1.1 | 修正：`gh mcp` 指令不存在，改用 GitHub 官方遠端 MCP Server (`api.githubcopilot.com/mcp/`) |
