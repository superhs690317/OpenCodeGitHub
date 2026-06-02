## GitHub 專案操作指引

### 帳號資訊

- GitHub 帳號：superhs690317
- Git 使用者名稱：Handsome
- Git 使用者 email：superhs690317@gmail.com
- 預設遠端通訊協定：https
- 測試 repo：github-test
- GitHub Pages 預設網址：https://superhs690317.github.io/github-test/

### 常用操作

1. **推送專案到 GitHub**
   - 檢查 `git status` → `git add` → `git commit` → `git push`
   - 若尚未設定 remote：`gh repo create <name> --public --source=. --push`

2. **開啟 GitHub Pages**
   - `gh api repos/{owner}/{repo}/pages -X POST -f "source[branch]=master" -f "source[path]=/"`

3. **檢查 GitHub CLI 狀態**
   - `gh auth status`

### 環境變數

- 啟動 OpenCode 前須設定 `GITHUB_TOKEN`：`$env:GITHUB_TOKEN = gh auth token`
- 建議將此行加入 `$PROFILE` 以自動載入

### MCP 設定

- GitHub MCP 已設定於根目錄 `opencode.json`
- 使用 GitHub 官方遠端 MCP Server：`https://api.githubcopilot.com/mcp/`
- 需透過 `Authorization: Bearer {env:GITHUB_TOKEN}` 進行驗證
- 設定了 `"oauth": false` 避免衝突

### 注意事項

- 不要將 token、密碼寫入任何檔案或 AGENTS.md
- commit 前先確認 `git status` 只包含預期異動
- 操作私有 repo 前需確認 token scope 包含 `repo`
