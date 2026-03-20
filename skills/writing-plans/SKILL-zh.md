---
name: writing-plans
description: 當您擁有多步驟任務的規格或需求時使用，在接觸程式碼之前
---

# 撰寫計畫

## 概述

撰寫全面的實作計畫，假設工程師對我們的程式碼庫毫無了解且品味存疑。記錄他們需要知道的一切：每個任務要修改哪些檔案、程式碼、測試、他們可能需要查閱的文件，以及如何測試。以小而可執行的任務形式呈現整個計畫。DRY。YAGNI。TDD。頻繁提交。

假設他們是熟練的開發者，但對我們的工具集或問題領域幾乎一無所知。假設他們不太懂好的測試設計。

**開始時宣告：** "I'm using the writing-plans skill to create the implementation plan."

**背景：** 這應該在專用 worktree（由 brainstorming skill 建立）中執行。

**計畫儲存至：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （使用者對計畫位置的偏好優先於此預設值）

## 範圍檢查

如果規格涵蓋多個獨立子系統，應在腦力激盪期間拆分為子專案規格。如果未拆分，建議將其拆分為獨立計畫 — 每個子系統一份。每個計畫應獨立產生可運作、可測試的軟體。

## 檔案結構

在定義任務之前，先規劃哪些檔案將被建立或修改，以及每個檔案的職責。這是鎖定分解決策的地方。

- 設計具有清晰邊界和良好定義介面的單元。每個檔案應有一個清晰的職責。
- 您對能一次納入上下文的程式碼推理最佳，且當檔案聚焦時您的編輯更可靠。偏好更小、聚焦的檔案，而非做太多事的大型檔案。
- 一起改變的檔案應住在一起。按職責分割，而非按技術層次。
- 在現有程式碼庫中，遵循既有模式。如果程式碼庫使用大型檔案，不要單方面重構 — 但如果您正在修改的檔案已變得難以管理，在計畫中包含拆分是合理的。

此結構為任務分解提供依據。每個任務應產生獨立合理的自包含變更。

## 小型任務粒度

**每個步驟是一個動作（2-5 分鐘）：**
- 「撰寫失敗的測試」— 步驟
- 「執行它以確保它失敗」— 步驟
- 「實作使測試通過的最小程式碼」— 步驟
- 「執行測試並確保它們通過」— 步驟
- 「提交」— 步驟

## 計畫文件標題

**每個計畫必須以此標題開始：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## 任務結構

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 注意事項
- 始終使用確切的檔案路徑
- 計畫中包含完整程式碼（而非「新增驗證」）
- 帶有預期輸出的確切命令
- 使用 @ 語法參照相關 skill
- DRY、YAGNI、TDD、頻繁提交

## 計畫審查循環

撰寫完整計畫後：

1. 派遣一個計畫文件審查者子代理（參見 plan-document-reviewer-prompt.md），提供精心準備的審查上下文 — 絕對不要使用您的會話歷史記錄。這讓審查者專注於計畫，而非您的思考過程。
   - 提供：計畫文件路徑、規格文件路徑
2. 如果 ❌ 發現問題：修復問題，重新派遣審查者審查整個計畫
3. 如果 ✅ 已核准：進入執行交接

**審查循環指引：**
- 由撰寫計畫的同一代理修復（保留上下文）
- 如果循環超過 3 次迭代，提交給人類尋求指導
- 審查者僅供建議 — 如果您認為回饋有誤，請解釋您的不同意見

## 執行交接

儲存計畫後，提供執行選擇：

**「計畫已完成並儲存至 `docs/superpowers/plans/<filename>.md`。兩種執行選項：**

**1. 子代理驅動（推薦）** — 我為每個任務派遣一個新鮮的子代理，在任務之間審查，快速迭代

**2. 內聯執行** — 在本次會話中使用 executing-plans 執行任務，批次執行並設有檢查點

**您想選哪種方式？」**

**如果選擇子代理驅動：**
- **必要子技能：** 使用 superpowers:subagent-driven-development
- 每個任務使用新鮮子代理 + 兩階段審查

**如果選擇內聯執行：**
- **必要子技能：** 使用 superpowers:executing-plans
- 帶有審查檢查點的批次執行
