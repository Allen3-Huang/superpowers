---
name: finishing-a-development-branch
description: 當實作完成、所有測試通過，且需要決定如何整合工作時使用——透過呈現合併、PR 或清理的結構化選項，引導完成開發工作
---

# 完成開發分支

## 概覽

透過呈現清晰選項並處理所選工作流程，引導完成開發工作。

**核心原則：** 驗證測試 → 呈現選項 → 執行選擇 → 清理。

**開始時宣告：** "我正在使用 finishing-a-development-branch 技能來完成這項工作。"

## 流程

### 步驟 1：驗證測試

**在呈現選項之前，先驗證測試是否通過：**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**若測試失敗：**
```
測試失敗（<N> 個失敗）。完成前必須修復：

[顯示失敗內容]

在測試通過之前無法進行合併/PR。
```

停止。不要繼續執行步驟 2。

**若測試通過：** 繼續執行步驟 2。

### 步驟 2：確定基礎分支

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或詢問：「此分支從 main 分出——是否正確？」

### 步驟 3：呈現選項

精確呈現以下 4 個選項：

```
實作完成。您希望如何處理？

1. 在本地合併回 <base-branch>
2. 推送並建立 Pull Request
3. 保留分支現狀（稍後自行處理）
4. 捨棄此工作

請選擇哪個選項？
```

**不要加入說明**——保持選項簡潔。

### 步驟 4：執行選擇

#### 選項 1：本地合併

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

然後：清理工作樹（步驟 5）

#### 選項 2：推送並建立 PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然後：清理工作樹（步驟 5）

#### 選項 3：保留現狀

回報：「保留分支 <name>。工作樹保存於 <path>。」

**不要清理工作樹。**

#### 選項 4：捨棄

**先確認：**
```
這將永久刪除：
- 分支 <name>
- 所有提交：<commit-list>
- 位於 <path> 的工作樹

輸入 'discard' 以確認。
```

等待確切的確認輸入。

若已確認：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然後：清理工作樹（步驟 5）

### 步驟 5：清理工作樹

**針對選項 1、2、4：**

檢查是否在工作樹中：
```bash
git worktree list | grep $(git branch --show-current)
```

若是：
```bash
git worktree remove <worktree-path>
```

**針對選項 3：** 保留工作樹。

## 快速參考

| 選項 | 合併 | 推送 | 保留工作樹 | 清理分支 |
|------|------|------|------------|----------|
| 1. 本地合併 | ✓ | - | - | ✓ |
| 2. 建立 PR | - | ✓ | ✓ | - |
| 3. 保留現狀 | - | - | ✓ | - |
| 4. 捨棄 | - | - | - | ✓（強制） |

## 常見錯誤

**跳過測試驗證**
- **問題：** 合併損壞的程式碼，建立失敗的 PR
- **修正：** 在提供選項之前，始終驗證測試

**開放式問題**
- **問題：** 「接下來我該怎麼做？」→ 含糊不清
- **修正：** 精確呈現 4 個結構化選項

**自動清理工作樹**
- **問題：** 在可能仍需要時移除工作樹（選項 2、3）
- **修正：** 僅在選項 1 和 4 時清理

**捨棄前未確認**
- **問題：** 意外刪除工作
- **修正：** 要求輸入「discard」確認

## 紅旗警示

**絕對不要：**
- 在測試失敗時繼續進行
- 在未驗證結果測試的情況下合併
- 未經確認就刪除工作
- 未經明確要求就強制推送

**始終：**
- 在提供選項之前驗證測試
- 精確呈現 4 個選項
- 針對選項 4 取得輸入確認
- 僅在選項 1 和 4 時清理工作樹

## 整合

**由以下技能呼叫：**
- **subagent-driven-development**（步驟 7）——所有任務完成後
- **executing-plans**（步驟 5）——所有批次完成後

**與以下技能搭配：**
- **using-git-worktrees**——清理由該技能建立的工作樹
