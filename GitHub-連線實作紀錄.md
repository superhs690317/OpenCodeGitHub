# GitHub 連線實作紀錄

## 文件資訊

- 專案名稱：`CodexGitHub`
- 執行日期：2026-05-10
- 執行環境：Windows PowerShell / Codex Desktop
- 參考文件：`02-連接-GitHub.md`

## 目標

依照 `02-連接-GitHub.md` 的內容，逐步完成以下事項：

1. 檢查本機 Git 與 GitHub CLI。
2. 驗證 Git 使用者資訊。
3. 確認 GitHub CLI 登入狀態。
4. 驗證 Codex Desktop 的 GitHub connector。
5. 建立或重用測試 repo，實際推送測試頁。
6. 啟用 GitHub Pages 並確認站台網址。
7. 將整個流程整理成可追蹤文件，上傳至 GitHub。

## 執行前狀態

- 工作目錄：`C:\Users\super\Documents\Codex\GitHub`
- 指定教學檔存在：`02-連接-GitHub.md`
- 此資料夾本身不是獨立 Git repo，而是位於上層 repo `CodexEnvironmentSetting` 之下。
- GitHub 帳號中已存在公開測試 repo：`superhs690317/github-test`

## 步驟與結果

### 步驟 0：環境檢查

執行結果如下：

- `git --version`
  - 結果：`git version 2.53.0.windows.2`
- `gh --version`
  - 初次執行失敗，錯誤為：
  - `failed to read configuration: open C:\Users\super\AppData\Roaming\GitHub CLI\config.yml: Access is denied.`
- `git config --global user.name`
  - 結果：`Handsome`
- `git config --global user.email`
  - 結果：`superhs690317@gmail.com`

判讀：

- Git 已安裝完成。
- Git 使用者資訊完整。
- `gh` 本身已安裝，但 Codex 執行時一開始沒有權限讀取 GitHub CLI 設定檔。

### 步驟 1：確認 GitHub CLI 登入狀態

在允許讀取 GitHub CLI 設定後重新檢查：

- `gh --version`
  - 結果：`gh version 2.89.0 (2026-03-26)`
- `gh auth status`
  - 帳號：`superhs690317`
  - Active account：`true`
  - Git operations protocol：`https`
  - Token scopes：`gist`, `read:org`, `repo`, `workflow`

判讀：

- GitHub CLI 已登入，不需要再跑 `gh auth login --web --git-protocol https`。
- 權限包含 `repo`，可支援建立、推送、修改 repo。

### 步驟 2：驗證 Codex Desktop GitHub connector

使用 Codex GitHub connector 驗證結果：

- 可讀取已安裝帳號：`superhs690317`
- 可讀取 installation 資訊
- 可列出可存取 repo
- 實測可見 repo 包含：
  - `superhs690317/CodexNotebookLm`
  - `superhs690317/LinkFirebase`
  - `superhs690317/LinkOllama`
  - `superhs690317/ClaudeCowork-LinkGemini`
  - `superhs690317/github-test`

判讀：

- `gh` CLI 與 Codex connector 都已連通。
- 本機推送與對話內讀取 GitHub 內容兩條路徑都可用。

### 步驟 3：重用既有測試 repo

因 GitHub 上已存在 `superhs690317/github-test`，所以未照教學新建同名 repo，而是改為重用既有 repo。

執行前確認：

- `gh repo view superhs690317/github-test --json name,visibility,url,defaultBranchRef`
- 結果：
  - repo 名稱：`github-test`
  - 可見性：`PUBLIC`
  - 預設分支：`master`

### 步驟 4：取得測試 repo 本機副本

第一次嘗試：

- `git clone https://github.com/superhs690317/github-test.git`
- 失敗原因：
  - `schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS (0x8009030e)`

改用已登入成功的 GitHub CLI：

- `gh repo clone superhs690317/github-test C:\Users\super\Documents\Codex\GitHub\github-test-gh`
- 結果：成功

判讀：

- Git 直接走 Windows 憑證通道時發生憑證取得失敗。
- 改走 `gh repo clone` 可繞過這個問題，屬於此環境可用的穩定替代路徑。

### 步驟 5：檢查 clone 後狀態

檢查結果：

- repo 內只有 `README.md`
- 尚未存在 `index.html`

另外發現：

- `git status` 出現 `detected dubious ownership`
- 原因是 sandbox 執行身份與資料夾擁有者不同

處理方式：

- 後續 Git 指令都改用：
  - `git -c safe.directory='C:/Users/super/Documents/Codex/GitHub/github-test-gh' -C ...`

這種寫法只對本次命令生效，不修改全域 Git 安全設定。

### 步驟 6：建立測試頁並提交

建立的 `index.html` 內容如下：

```html
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
```

提交過程：

- `git add index.html`
- `git commit -m "建立 GitHub 連線測試頁"`

結果：

- commit 成功
- commit short SHA：`c76f018`

### 步驟 7：推送到 GitHub

執行：

- `git push origin master`

結果：

- push 成功
- 遠端更新：`c989c07..c76f018`

判讀：

- 即使本機直接 `git clone` 曾因憑證問題失敗，既有遠端與已登入狀態下的 `git push` 仍可正常完成。

### 步驟 8：建立 GitHub Pages

第一次查詢：

- `gh api repos/superhs690317/github-test/pages`
- 結果：`404 Not Found`

第一次建立方式：

- `gh api repos/superhs690317/github-test/pages -X POST -f build_type=legacy -f source.branch=master -f source.path=/`
- 結果：`422`
- 錯誤訊息：`The gh-pages branch must exist before GitHub Pages can be built.`

後續查核 GitHub 官方 Pages REST API 文件後，改用目前正確的表單欄位格式：

- `gh api repos/superhs690317/github-test/pages -X POST -f "source[branch]=master" -f "source[path]=/"`

結果：

- Pages 建立成功
- 站台網址：`https://superhs690317.github.io/github-test/`
- source branch：`master`
- source path：`/`
- HTTPS：啟用

### 步驟 9：檢查 Pages build 狀態

執行：

- `gh api repos/superhs690317/github-test/pages/builds/latest`

查詢當下結果：

- status：`building`
- build id：`991860035`
- commit：`c76f0189ddfeae7b58d2c801cade73ecc655a905`

判讀：

- Pages 站台已建立成功。
- 查詢當下仍在部署中，屬於正常現象，通常需再等待幾十秒到數分鐘。

## 成功驗證項目

- Git 已可正常使用。
- GitHub CLI 已安裝且登入成功。
- Git 全域姓名與 email 已正確設定。
- Codex Desktop GitHub connector 已可讀取帳號、installation 與 repo。
- 測試 repo 已成功更新並推送。
- GitHub Pages 已建立完成並取得固定網址。

## 本次踩坑與處理方式

### 1. GitHub CLI 設定檔權限被擋

- 現象：
  - `failed to read configuration: open ... config.yml: Access is denied.`
- 處理：
  - 允許 Codex 讀取 GitHub CLI 設定位置後重試。

### 2. `git clone` 遇到 Windows 憑證錯誤

- 現象：
  - `SEC_E_NO_CREDENTIALS`
- 處理：
  - 改用 `gh repo clone`。

### 3. clone 後出現 `dubious ownership`

- 現象：
  - Git 認定目前執行身份與資料夾擁有者不同。
- 處理：
  - 在單次命令中加入 `-c safe.directory=...`。

### 4. GitHub Pages API 舊寫法導向錯誤模式

- 現象：
  - 傳 `build_type=legacy` 後被要求先有 `gh-pages` branch。
- 處理：
  - 依 GitHub 官方目前 API 格式改送 `source[branch]` 與 `source[path]`。

## 最終成果

- 測試 repo：`https://github.com/superhs690317/github-test`
- GitHub Pages：`https://superhs690317.github.io/github-test/`
- 本次文件 repo：預計建立為 `superhs690317/CodexGitHub`

## 建議後續用法

完成這次設定後，後續可直接對 Codex 下列指令：

- 幫我把這個專案推到 GitHub
- 幫我建立新的 GitHub repo
- 幫我把這個靜態網頁開成 GitHub Pages
- 幫我檢查目前 repo 狀態並同步遠端

## 附註

- 本次文件是依實際執行結果撰寫，不是單純轉述教學內容。
- 若未來 GitHub Pages API 再調整，建議優先以 GitHub 官方 REST API 文件為準。
