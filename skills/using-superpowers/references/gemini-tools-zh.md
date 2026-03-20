# Gemini CLI 工具對應

技能使用 Claude Code 的工具名稱。當你在技能中遇到這些名稱時，請使用你平台的對應工具：

| 技能中的參考 | Gemini CLI 對應工具 |
|-----------------|----------------------|
| `Read`（讀取檔案） | `read_file` |
| `Write`（建立檔案） | `write_file` |
| `Edit`（編輯檔案） | `replace` |
| `Bash`（執行命令） | `run_shell_command` |
| `Grep`（搜尋檔案內容） | `grep_search` |
| `Glob`（依名稱搜尋檔案） | `glob` |
| `TodoWrite`（任務追蹤） | `write_todos` |
| `Skill` 工具（呼叫技能） | `activate_skill` |
| `WebSearch` | `google_web_search` |
| `WebFetch` | `web_fetch` |
| `Task` 工具（派遣子代理） | 無對應工具——Gemini CLI 不支援子代理 |

## 不支援子代理

Gemini CLI 沒有等同於 Claude Code `Task` 工具的功能。依賴子代理派遣的技能（`subagent-driven-development`、`dispatching-parallel-agents`）將透過 `executing-plans` 退回到單一對話執行模式。

## Gemini CLI 的額外工具

以下工具在 Gemini CLI 中可用，但在 Claude Code 中沒有對應工具：

| 工具 | 用途 |
|------|---------|
| `list_directory` | 列出檔案和子目錄 |
| `save_memory` | 跨對話將事實持久化至 GEMINI.md |
| `ask_user` | 向使用者請求結構化輸入 |
| `tracker_create_task` | 豐富的任務管理（建立、更新、列出、視覺化） |
| `enter_plan_mode` / `exit_plan_mode` | 在進行變更前切換至唯讀研究模式 |
