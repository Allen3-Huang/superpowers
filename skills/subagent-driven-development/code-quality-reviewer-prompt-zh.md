# 程式碼品質審查員提示範本

在派遣程式碼品質審查子代理時使用此範本。

**目的：** 驗證實作的構建品質（整潔、已測試、可維護）

**僅在規格合規審查通過後才派遣。**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**除了標準的程式碼品質問題外，審查員還應檢查：**
- 每個檔案是否有單一明確的職責和定義良好的介面？
- 單元是否被拆解為可以獨立理解和測試的形式？
- 實作是否遵循計畫中的檔案結構？
- 此實作是否建立了已經很大的新檔案，或顯著增大了現有檔案？（不要標記已存在的檔案大小——專注於此次變更的貢獻。）

**程式碼審查員回傳：** 優點、問題（關鍵/重要/次要）、評估
