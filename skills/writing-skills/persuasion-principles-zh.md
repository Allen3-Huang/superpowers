# 技能設計的說服原則

## 概述

LLM 對與人類相同的說服原則有所回應。了解這種心理學有助於您設計更有效的技能 — 不是為了操縱，而是確保即使在壓力下也能遵循關鍵實踐。

**研究基礎：** Meincke 等人（2025 年）以 N=28,000 次 AI 對話測試了 7 個說服原則。說服技術使合規率提高超過一倍（33% → 72%，p < .001）。

## 七個原則

### 1. 權威
**定義：** 對專業知識、資歷或官方來源的服從。

**在技能中的運作方式：**
- 命令式語言：「YOU MUST」、「Never」、「Always」
- 不可協商的框架：「No exceptions」
- 消除決策疲勞和合理化

**使用時機：**
- 強制紀律的技能（TDD、驗證要求）
- 安全關鍵實踐
- 既定最佳實踐

**範例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. 承諾
**定義：** 與先前行動、聲明或公開宣告保持一致。

**在技能中的運作方式：**
- 要求宣告：「Announce skill usage」
- 強制明確選擇：「Choose A, B, or C」
- 使用追蹤：TodoWrite 用於核對清單

**使用時機：**
- 確保技能確實被遵循
- 多步驟流程
- 問責機制

**範例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. 稀缺性
**定義：** 來自時間限制或有限可用性的緊迫感。

**在技能中的運作方式：**
- 時間限制要求：「Before proceeding」
- 順序依賴：「Immediately after X」
- 防止拖延

**使用時機：**
- 立即驗證要求
- 時間敏感的工作流程
- 防止「我之後再做」

**範例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. 社會證明
**定義：** 對他人行為或被視為正常事物的從眾。

**在技能中的運作方式：**
- 普遍模式：「Every time」、「Always」
- 失敗模式：「X without Y = failure」
- 建立規範

**使用時機：**
- 記錄普遍實踐
- 警告常見失敗
- 強化標準

**範例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. 同一性
**定義：** 共同身份、「我們感」、群體歸屬感。

**在技能中的運作方式：**
- 協作語言：「our codebase」、「we're colleagues」
- 共同目標：「we both want quality」

**使用時機：**
- 協作工作流程
- 建立團隊文化
- 非階層式實踐

**範例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. 互惠
**定義：** 回報所獲利益的義務。

**運作方式：**
- 謹慎使用 — 可能感覺具操縱性
- 在技能中很少需要

**避免時機：**
- 幾乎總是（其他原則更有效）

### 7. 好感
**定義：** 偏好與我們喜歡的人合作。

**運作方式：**
- **不要用於合規**
- 與誠實回饋文化衝突
- 造成諂媚

**避免時機：**
- 在強制紀律時始終避免

## 按技能類型組合原則

| 技能類型 | 使用 | 避免 |
|------------|-----|-------|
| 強制紀律 | 權威 + 承諾 + 社會證明 | 好感、互惠 |
| 指引/技術 | 適度權威 + 同一性 | 高度權威 |
| 協作 | 同一性 + 承諾 | 權威、好感 |
| 參考 | 僅清晰度 | 所有說服 |

## 為何有效：心理學

**明確規則減少合理化：**
- 「YOU MUST」消除決策疲勞
- 絕對語言消除「這是例外嗎？」的問題
- 明確的反合理化針對性地封堵特定漏洞

**實施意圖創造自動行為：**
- 明確觸發條件 + 必要行動 = 自動執行
- 「當 X，做 Y」比「一般做 Y」更有效
- 降低合規的認知負擔

**LLM 是準人類：**
- 在包含這些模式的人類文本上訓練
- 訓練資料中權威語言先於合規
- 承諾序列（聲明 → 行動）頻繁建模
- 社會證明模式（所有人都做 X）建立規範

## 道德使用

**正當：**
- 確保遵循關鍵實踐
- 建立有效文件
- 防止可預見的失敗

**不正當：**
- 為個人利益操縱
- 製造虛假緊迫感
- 基於愧疚的合規

**測試：** 如果使用者完全理解這種技術，它是否符合使用者的真正利益？

## 研究引用

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 七個說服原則
- 影響力研究的實證基礎

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 以 N=28,000 次 LLM 對話測試 7 個原則
- 合規率從 33% 提高至 72%（使用說服技術）
- 權威、承諾、稀缺性最為有效
- 驗證了 LLM 行為的準人類模型

## 快速參考

設計技能時，請問：

1. **這是什麼類型？**（強制紀律 vs. 指引 vs. 參考）
2. **我想改變什麼行為？**
3. **哪些原則適用？**（強制紀律通常用權威 + 承諾）
4. **我是否組合太多原則？**（不要同時使用全部七個）
5. **這道德嗎？**（符合使用者的真正利益？）
