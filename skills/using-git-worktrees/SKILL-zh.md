---
name: using-git-worktrees
description: 在開始需要與目前工作區隔離的功能開發時，或在執行實作計劃之前使用——建立具有智慧目錄選擇和安全驗證的隔離 git worktree
---

# 使用 Git Worktrees

## 概述

Git worktree 建立共享相同儲存庫的隔離工作區，允許同時在多個分支上工作而無需切換。

**核心原則：** 系統化的目錄選擇 + 安全驗證 = 可靠的隔離。

**在開始時宣告：** 「我正在使用 using-git-worktrees 技能來建立隔離的工作區。」

## 目錄選擇流程

依照以下優先順序：

### 1. 檢查現有目錄

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**如果找到：** 使用該目錄。如果兩者都存在，`.worktrees` 優先。

### 2. 檢查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**如果有指定偏好：** 直接使用，無需詢問。

### 3. 詢問使用者

如果沒有目錄存在且 CLAUDE.md 中沒有偏好設定：

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## 安全驗證

### 針對專案本地目錄（.worktrees 或 worktrees）

**在建立 worktree 之前，必須驗證目錄已被忽略：**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**如果未被忽略：**

根據 Jesse 的規則「立即修復損壞的東西」：
1. 在 .gitignore 中添加適當的行
2. 提交該變更
3. 繼續建立 worktree

**為什麼至關重要：** 防止意外將 worktree 內容提交至儲存庫。

### 針對全局目錄（~/.config/superpowers/worktrees）

無需驗證 .gitignore——完全在專案之外。

## 建立步驟

### 1. 偵測專案名稱

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 建立 Worktree

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 執行專案設定

自動偵測並執行適當的設定：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 驗證乾淨的基準線

執行測試以確保 worktree 從乾淨狀態開始：

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**如果測試失敗：** 回報失敗情況，詢問是否繼續或調查。

**如果測試通過：** 回報已就緒。

### 5. 回報位置

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 快速參考

| 情況 | 動作 |
|-----------|--------|
| `.worktrees/` 存在 | 使用它（驗證已忽略） |
| `worktrees/` 存在 | 使用它（驗證已忽略） |
| 兩者都存在 | 使用 `.worktrees/` |
| 兩者都不存在 | 檢查 CLAUDE.md → 詢問使用者 |
| 目錄未被忽略 | 添加至 .gitignore + 提交 |
| 基準線測試失敗 | 回報失敗 + 詢問 |
| 沒有 package.json/Cargo.toml | 跳過依賴安裝 |

## 常見錯誤

### 跳過忽略驗證

- **問題：** Worktree 內容被追蹤，污染 git 狀態
- **修正：** 在建立專案本地 worktree 之前，始終使用 `git check-ignore`

### 假設目錄位置

- **問題：** 造成不一致，違反專案慣例
- **修正：** 遵循優先順序：現有目錄 > CLAUDE.md > 詢問使用者

### 在測試失敗時繼續

- **問題：** 無法區分新錯誤與既有問題
- **修正：** 回報失敗，取得明確許可後再繼續

### 硬編碼設定命令

- **問題：** 在使用不同工具的專案上失效
- **修正：** 從專案檔案自動偵測（package.json 等）

## 範例工作流程

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 紅旗警示

**絕不：**
- 在未驗證已忽略的情況下建立 worktree（專案本地）
- 跳過基準線測試驗證
- 在未詢問的情況下帶著失敗的測試繼續
- 在不明確時假設目錄位置
- 跳過 CLAUDE.md 檢查

**始終：**
- 遵循目錄優先順序：現有目錄 > CLAUDE.md > 詢問使用者
- 驗證專案本地目錄已被忽略
- 自動偵測並執行專案設定
- 驗證乾淨的測試基準線

## 整合

**被以下技能呼叫：**
- **brainstorming**（第 4 階段）——當設計獲批准且實作跟進時為必要步驟
- **subagent-driven-development**——在執行任何任務之前為必要步驟
- **executing-plans**——在執行任何任務之前為必要步驟
- 任何需要隔離工作區的技能

**搭配使用：**
- **finishing-a-development-branch**——工作完成後進行清理的必要步驟
