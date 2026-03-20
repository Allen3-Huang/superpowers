---
name: requesting-code-review
description: 在完成任務、實作主要功能，或在合併之前用於驗證工作是否符合需求時使用
---

# 請求程式碼審查

派遣 superpowers:code-reviewer 子代理在問題擴散之前捕捉錯誤。審查者獲得精心設計的評估背景——而非您的工作階段歷史記錄。這使審查者專注於工作成果，而非您的思考過程，同時保留您自己的背景以繼續工作。

**核心原則：** 盡早審查，頻繁審查。

## 何時請求審查

**必要情況：**
- 在子代理驅動開發中的每個任務之後
- 完成主要功能之後
- 合併到 main 之前

**選擇性但有價值：**
- 當卡住時（全新視角）
- 重構之前（基準檢查）
- 修復複雜錯誤之後

## 如何請求

**1. 取得 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派遣 code-reviewer 子代理：**

使用 Task 工具搭配 superpowers:code-reviewer 類型，填入 `code-reviewer.md` 中的範本

**佔位符：**
- `{WHAT_WAS_IMPLEMENTED}` - 您剛建置的內容
- `{PLAN_OR_REQUIREMENTS}` - 應完成的事項
- `{BASE_SHA}` - 起始提交
- `{HEAD_SHA}` - 結束提交
- `{DESCRIPTION}` - 簡短摘要

**3. 根據回饋行動：**
- 立即修復「嚴重」問題
- 在繼續之前修復「重要」問題
- 記錄「次要」問題供後續處理
- 若審查者有誤則反駁（附理由）

## 範例

```
[剛完成任務 2：新增驗證函數]

您：讓我在繼續之前請求程式碼審查。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派遣 superpowers:code-reviewer 子代理]
  WHAT_WAS_IMPLEMENTED: 對話索引的驗證和修復函數
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的任務 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 新增了帶有 4 種問題類型的 verifyIndex() 和 repairIndex()

[子代理返回]：
  優點：乾淨的架構，真實的測試
  問題：
    重要：缺少進度指示器
    次要：報告間隔的魔術數字 (100)
  評估：可以繼續

您：[修復進度指示器]
[繼續執行任務 3]
```

## 與工作流程的整合

**子代理驅動開發：**
- 在每個任務之後審查
- 在問題積累之前捕捉它們
- 在繼續下一個任務之前修復

**執行計劃：**
- 在每批（3 個任務）之後審查
- 取得回饋，應用，繼續

**臨時開發：**
- 合併前審查
- 卡住時審查

## 紅旗警示

**絕對不要：**
- 因為「很簡單」而跳過審查
- 忽略「嚴重」問題
- 在未修復「重要」問題的情況下繼續
- 與有效的技術回饋爭辯

**若審查者有誤：**
- 以技術理由反駁
- 展示證明可運作的程式碼/測試
- 請求澄清

範本位於：requesting-code-review/code-reviewer.md
