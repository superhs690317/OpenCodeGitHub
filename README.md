# OpenCodeGitHub

這個專案記錄在 Windows 環境下，使用 **OpenCode CLI** 完成 GitHub 連線、權限驗證、測試 repo 推送與 GitHub Pages 建立的完整過程。

## 內容

- `02-連接-GitHub.md`
  - OpenCode 專用的 GitHub 連線操作教學與檢查清單。
- `OpenCode-連線GitHub實作紀錄.md`
  - 依教學逐步執行的實測紀錄、錯誤排查、結果與後續建議。

## 實測結果摘要

- Git 可正常使用。
- GitHub CLI 可正常使用，且帳號已登入。
- OpenCode 可透過 `gh` CLI 與 MCP 整合 GitHub 服務。
- 測試 repo `superhs690317/github-test` 已成功推送測試頁。
- GitHub Pages 已建立，網址為 `https://superhs690317.github.io/github-test/`。

## 備註

- 本專案的 `opencode.json` 與 `AGENTS.md` 提供 OpenCode CLI 在執行 GitHub 相關操作時的指引與設定。
- 所有操作以 OpenCode CLI 為核心，不再依賴 Codex Desktop 的 GUI connector。
