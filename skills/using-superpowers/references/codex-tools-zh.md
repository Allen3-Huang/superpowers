# Codex 工具對應

技能使用 Claude Code 的工具名稱。當你在技能中遇到這些名稱時，請使用你平台的對應工具：

| 技能中的參考 | Codex 對應工具 |
|-----------------|------------------|
| `Task` 工具（派遣子代理） | `spawn_agent` |
| 多個 `Task` 呼叫（並行） | 多個 `spawn_agent` 呼叫 |
| Task 回傳結果 | `wait` |
| Task 自動完成 | `close_agent` 以釋放槽位 |
| `TodoWrite`（任務追蹤） | `update_plan` |
| `Skill` 工具（呼叫技能） | 技能以原生方式載入——直接遵循指令即可 |
| `Read`、`Write`、`Edit`（檔案） | 使用你的原生檔案工具 |
| `Bash`（執行命令） | 使用你的原生 shell 工具 |

## 子代理派遣需要多代理支援

在你的 Codex 設定中添加（`~/.codex/config.toml`）：

```toml
[features]
multi_agent = true
```

這會啟用 `spawn_agent`、`wait` 和 `close_agent`，供 `dispatching-parallel-agents` 和 `subagent-driven-development` 等技能使用。
