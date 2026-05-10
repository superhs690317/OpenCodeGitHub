# Codex 懶人包 #02：連接 GitHub

> 版本：v0.3（Codex Desktop 版）
> 更新日期：2026-04-27

> 本懶人包可獨立執行：會先檢查 Git、GitHub CLI、登入狀態，再用網頁端登入完成 GitHub CLI 授權，接著引導連接 Codex Desktop 的 GitHub connector。

---

## 這個懶人包會幫你做什麼？

讓 Codex 可以幫你操作 GitHub，包括：

- 建立 GitHub repo
- 把 Codex 做好的網頁、教材、互動 HTML 推送上線
- 開啟 GitHub Pages，讓學生掃 QR Code 就能開啟
- 之後用自然語言完成 commit、push、Pages 發布
- 在 Codex 對話中讀取 GitHub repo、issue、PR 內容

> [!important]
> GitHub 連線不是只靠 Codex 本身完成，而是靠電腦上的 Git、GitHub CLI，以及 GitHub 帳號授權。Codex 的角色是幫你檢查、執行、判讀錯誤、一步一步帶你完成。

> [!tip]
> 完整連線分成兩段：`gh` CLI 負責本機 push / 建 repo / 開 Pages；Codex Desktop 的 GitHub connector 負責讓 Codex 在對話裡讀取 GitHub repo、issue、PR。

---

## 適用情境

本懶人包以 **Codex Desktop app（Windows）** 為主線，並使用 PowerShell 指令。

也可套用到：

| 工具 | 說明 |
|------|------|
| Codex Desktop app | 推薦，新手照本懶人包走 |
| Codex IDE 擴充 | 可沿用同樣的 Git / gh 設定 |
| Codex CLI | 可沿用同樣指令，但操作畫面不同 |

---

## 先備條件

- [ ] Codex Desktop app 已安裝並能正常對話
- [ ] 有 GitHub 帳號；如果沒有，請先到 <https://github.com/signup> 註冊
- [ ] 電腦有網路連線
- [ ] 願意在瀏覽器完成一次 GitHub 授權

---

## 請 Codex 幫我執行以下步驟

> [!warning]
> 以下是給 Codex 讀的操作流程。遇到需要登入、授權、輸入驗證碼的地方，Codex 會停下來請使用者手動完成。

---

## 步驟零：環境檢查

請 Codex 先檢查目前環境，不要直接假設工具都已安裝。

### 0.1 檢查 Git

```powershell
git --version
```

成功時會看到類似：

```text
git version 2.53.0.windows.1
```

如果沒有安裝 Git：

```powershell
winget install --id Git.Git --accept-source-agreements --accept-package-agreements
```

安裝完若 Codex 還是找不到 `git`，請重新啟動 Codex Desktop app。

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

Windows 常見安裝位置：

```text
C:\Program Files\GitHub CLI\gh.exe
```

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

> [!tip]
> 如果你只是要操作公開 repo，有些權限可以比較少；但如果 Codex 要幫你建立、推送、修改私有 repo，建議確認 scope 有 `repo`。

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
6. 回到 Codex / PowerShell，確認登入完成。

登入後再檢查一次：

```powershell
gh auth status
```

> [!important]
> 這次實測成功的路線就是網頁端登入。不要卡在純終端機輸入帳密，GitHub 現在通常會要求走瀏覽器授權。

---

## 步驟三：連接 Codex Desktop 的 GitHub connector

完成 `gh` CLI 登入後，建議再把 Codex Desktop 內建的 GitHub connector 也連起來。

這一步不是 PowerShell 指令，而是在 Codex Desktop app 的設定介面完成。

### 3.1 打開 Codex 的連接器設定

請使用者在 Codex Desktop app 裡依序尋找類似位置：

```text
Settings / 設定
Connectors / Apps / Integrations
GitHub
Connect / Sign in / Install
```

不同版本介面文字可能略有差異，重點是找到 GitHub 連接器或 GitHub App。

### 3.2 用瀏覽器登入並授權

Codex 會開啟 GitHub 網頁授權流程。

請使用者依畫面完成：

1. 登入 GitHub 帳號。
2. 選擇要授權的帳號或 organization。
3. 選擇允許 Codex 存取的 repo 範圍。
4. 點選 Install / Authorize / Connect。
5. 回到 Codex Desktop app。

> [!tip]
> 新手建議先授權自己的個人帳號，repo 範圍可依需求選「全部 repo」或「只選指定 repo」。如果之後 Codex 找不到某個 repo，通常是 GitHub App 沒有被授權到那個 repo。

### 3.3 請 Codex 驗證 connector 是否成功

連接完成後，請使用者在 Codex 對話中輸入：

```text
幫我確認 GitHub connector 是否已連接，並列出我可以存取的 GitHub 帳號與幾個 repo。
```

成功時，Codex 應該能看到：

| 檢查項目 | 成功狀態 |
|----------|----------|
| GitHub 帳號 | 看得到使用者帳號 |
| installation | 看得到 GitHub App 安裝資訊 |
| repo 權限 | 看得到 repo，並能辨識 pull / push / admin 等權限 |

### 3.4 connector 失敗時怎麼辦

如果 Codex 顯示：

```text
accounts: []
installations: []
repositories: []
```

通常代表 GitHub connector 還沒有真的連上。

處理方式：

1. 回到 Codex Desktop 的 GitHub connector 設定頁。
2. 重新 Connect / Sign in / Install。
3. 確認瀏覽器登入的是正確 GitHub 帳號。
4. 確認 GitHub App 有授權到需要的 repo。
5. 回到 Codex，重新請它檢查 connector。

> [!important]
> `gh auth status` 成功，只代表本機 GitHub CLI 登入成功；不代表 Codex Desktop connector 一定已連接。兩個都成功，才是完整 GitHub 工作流。

---

## 步驟四：建立測試 repo 驗證

為了確認 Codex 真的能寫入 GitHub，建議建立一個測試 repo。

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
gh api repos/{owner}/github-test/pages -X POST -f build_type=legacy -f source.branch=main -f source.path=/
```

如果上面指令失敗，可能是 repo 還沒完全建立完成。等 30 秒後再試一次，或請 Codex 幫你到 GitHub 網頁檢查 Pages 設定。

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

測試完成後，請 Codex 詢問使用者：

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

| 你說的話 | Codex 會做的事 |
|----------|----------------|
| 「幫我把這個專案推到 GitHub」 | 檢查 diff、commit、push |
| 「幫我建立 GitHub repo」 | 建 repo、設定 remote、推送 |
| 「幫我把這個網頁上線」 | 推送 repo、設定 GitHub Pages、回報網址 |
| 「幫我同步 GitHub」 | 檢查目前 repo 狀態，需要時 commit / push |
| 「收工」 | 若已設定收工 skill，會一起檢查 Obsidian、AGENTS.md、GitHub |

---

## Codex 環境下的注意事項

### 1. Codex 可能需要授權才能讀寫 GitHub CLI 設定

在 Codex Desktop app 裡，有時直接執行 `gh --version` 或 `gh auth status` 會遇到類似：

```text
failed to read configuration: open C:\Users\<你>\AppData\Roaming\GitHub CLI\config.yml: Access is denied.
```

這通常不是 GitHub CLI 壞掉，而是 Codex 需要你允許它讀取 GitHub CLI 的設定位置。

處理方式：

1. 讓 Codex 重新執行檢查，並同意授權。
2. 授權後再跑：

```powershell
gh auth status
```

3. 看到登入帳號與 `repo` scope 後再繼續。

### 2. Codex App 連接器與 GitHub CLI 是兩件事

Codex 裡可能有 GitHub connector / app，也可能有本機 `gh` CLI。

| 類型 | 用途 |
|------|------|
| GitHub connector | 讓 Codex 在對話中查 repo、issue、PR 等 GitHub 資料 |
| GitHub CLI (`gh`) | 讓 Codex 在你的電腦上建立 repo、commit、push、開 Pages |

如果只是要讓 Codex 在本機專案裡幫你 commit / push，`gh` CLI 是最直接的路線。

如果要在對話中直接讀 GitHub 內容、查 PR、看 issue，則要設定 Codex 的 GitHub connector。完整新手流程建議兩個都設定。

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

token、密碼、一次性驗證碼都不要寫進 repo 或 Obsidian 對外筆記。

---

## 踩坑筆記

| 狀況 | 原因 | 解法 |
|------|------|------|
| `gh --version` 顯示 Access is denied | Codex 被擋在 GitHub CLI 設定檔外 | 允許 Codex 讀取後重跑 |
| `gh auth status` 沒有登入 | 尚未完成 GitHub 授權 | 執行 `gh auth login --web --git-protocol https` |
| Codex connector 顯示 `accounts: []` | Codex Desktop 尚未連接 GitHub App | 到 Codex 設定裡重新 Connect GitHub |
| connector 看得到帳號但找不到 repo | GitHub App 沒有授權該 repo | 到 GitHub App installation 設定補授權 repo |
| 瀏覽器沒有自動開 | 裝置登入頁沒被自動喚起 | 手動開 `https://github.com/login/device` |
| push 被拒絕 | 權限不足或不是正確帳號 | 重跑 `gh auth status`，確認帳號與 `repo` scope |
| GitHub Pages 404 | Pages 尚未部署完成 | 等 1 到 3 分鐘再重整 |
| Codex 可以讀 repo 但不能 push | connector 可讀不等於本機 git 有權限 | 檢查 `gh auth status` 與 git remote |

---

## 更新紀錄

| 日期 | 版本 | 更新內容 |
|------|------|----------|
| 2026-04-26 | v0.1 | Codex 初版 |
| 2026-04-27 | v0.2 | 改成 Codex Desktop 主線，補上網頁端登入、PowerShell 指令、實測踩坑 |
| 2026-04-27 | v0.3 | 補上 Codex Desktop GitHub connector 的登入、授權與驗證流程 |
