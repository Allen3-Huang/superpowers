---
name: verification-before-completion
description: 當即將聲稱工作已完成、已修復或已通過測試時使用，在提交或建立 PR 之前 — 需要執行驗證命令並確認輸出，然後才能提出任何成功聲明；主張之前必須先有證據
---

# 完成前的驗證

## 概述

在未驗證的情況下聲稱工作已完成是不誠實的行為，而非效率的體現。

**核心原則：** 永遠先有證據，再做聲明。

**違反此規則的字面意義，就是違反此規則的精神。**

## 鐵律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

如果您在此訊息中尚未執行驗證命令，則無法聲稱它已通過。

## 關卡函數

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## 常見失敗情況

| 聲明 | 需要 | 不足以 |
|-------|----------|----------------|
| 測試通過 | 測試命令輸出：0 個失敗 | 上一次執行、「應該會通過」 |
| Linter 無誤 | Linter 輸出：0 個錯誤 | 部分檢查、外推 |
| 建置成功 | 建置命令：exit 0 | Linter 通過、日誌看起來正常 |
| 錯誤已修復 | 測試原始症狀：通過 | 程式碼已更改、假設已修復 |
| 回歸測試有效 | 紅綠循環已驗證 | 測試通過一次 |
| Agent 已完成 | VCS 差異顯示更改 | Agent 回報「成功」 |
| 需求已達成 | 逐行核對清單 | 測試通過 |

## 警示訊號 - 停止

- 使用「應該」、「可能」、「似乎」
- 在驗證前表達滿意（「太好了！」「完美！」「完成了！」等）
- 即將在未驗證的情況下 commit/push/PR
- 信任 Agent 的成功回報
- 依賴部分驗證
- 認為「就這一次」
- 疲憊且想要結束工作
- **任何在未執行驗證的情況下暗示成功的措辭**

## 防止合理化

| 藉口 | 現實 |
|--------|---------|
| 「現在應該可以運作」 | 執行驗證 |
| 「我有信心」 | 信心 ≠ 證據 |
| 「就這一次」 | 沒有例外 |
| 「Linter 已通過」 | Linter ≠ 編譯器 |
| 「Agent 說成功了」 | 獨立驗證 |
| 「我累了」 | 疲憊 ≠ 藉口 |
| 「部分檢查就夠了」 | 部分證明不了任何事 |
| 「用了不同措辭所以規則不適用」 | 精神優先於字面 |

## 關鍵模式

**測試：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**回歸測試（TDD 紅綠循環）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**建置：**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**需求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**Agent 委派：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## 為何重要

來自 24 次失敗記憶：
- 您的人類夥伴說「我不相信你」— 信任已破裂
- 未定義的函數被發布 — 會導致崩潰
- 缺少需求被發布 — 功能不完整
- 在虛假完成上浪費時間 → 重新引導 → 返工
- 違反：「誠實是核心價值。如果你撒謊，你將被取代。」

## 何時適用

**永遠在以下情況之前：**
- 任何形式的成功/完成聲明
- 任何表達滿意的行為
- 任何關於工作狀態的正面陳述
- 提交、建立 PR、完成任務
- 移至下一個任務
- 委派給 Agent

**規則適用於：**
- 確切短語
- 釋義和同義詞
- 成功的暗示
- 任何暗示完成/正確性的溝通

## 結論

**驗證沒有捷徑。**

執行命令。讀取輸出。然後聲明結果。

這是不可協商的。
