# 技能撰寫最佳實踐

> 學習如何撰寫 Claude 能有效發現和使用的技能。

好的技能簡潔、結構良好，並經過真實使用測試。本指南提供實際的撰寫決策，幫助您撰寫 Claude 能有效發現和使用的技能。

有關技能如何運作的概念背景，請參閱[技能概述](/en/docs/agents-and-tools/agent-skills/overview)。

## 核心原則

### 簡潔是關鍵

[上下文視窗](https://platform.claude.com/docs/en/build-with-claude/context-windows)是公共資源。您的技能與 Claude 需要知道的其他所有內容共享上下文視窗，包括：

* 系統提示
* 對話歷史
* 其他技能的元資料
* 您的實際請求

您技能中的每個 token 並非都有即時成本。啟動時，只有所有技能的元資料（名稱和描述）被預先載入。Claude 僅在技能變得相關時才讀取 SKILL.md，並僅在需要時才讀取其他檔案。然而，在 SKILL.md 中保持簡潔仍然重要：一旦 Claude 載入它，每個 token 都與對話歷史和其他上下文競爭。

**預設假設**：Claude 已經非常聰明

只添加 Claude 尚未擁有的上下文。對每條資訊提出挑戰：

* 「Claude 真的需要這個解釋嗎？」
* 「我能假設 Claude 知道這個嗎？」
* 「這段話值得它的 token 成本嗎？」

**好範例：簡潔**（約 50 個 token）：

````markdown  theme={null}
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**壞範例：過於冗長**（約 150 個 token）：

```markdown  theme={null}
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

簡潔版本假設 Claude 知道 PDF 是什麼以及函式庫如何運作。

### 設定適當的自由度

根據任務的脆弱性和可變性匹配具體程度。

**高自由度**（基於文字的指示）：

使用時機：

* 多種方法都有效
* 決策取決於上下文
* 啟發式方法指導方法

範例：

```markdown  theme={null}
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
```

**中等自由度**（帶參數的偽程式碼或腳本）：

使用時機：

* 存在首選模式
* 某些變化是可接受的
* 配置影響行為

範例：

````markdown  theme={null}
## Generate report

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
````

**低自由度**（具體腳本，少或無參數）：

使用時機：

* 操作脆弱且容易出錯
* 一致性至關重要
* 必須遵循特定順序

範例：

````markdown  theme={null}
## Database migration

Run exactly this script:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
````

**類比**：將 Claude 想象為在路徑上探索的機器人：

* **兩側有懸崖的窄橋**：只有一種安全的前進方式。提供具體的護欄和確切的指示（低自由度）。範例：必須按確切順序執行的資料庫遷移。
* **沒有危險的開闊田野**：許多路徑都能通往成功。給予大方向並信任 Claude 找到最佳路線（高自由度）。範例：上下文決定最佳方法的程式碼審查。

### 使用您計劃使用的所有模型進行測試

技能作為模型的附加項，因此有效性取決於底層模型。請使用您計劃使用的所有模型測試您的技能。

**按模型的測試考量**：

* **Claude Haiku**（快速、經濟）：技能是否提供足夠的指引？
* **Claude Sonnet**（均衡）：技能是否清晰且高效？
* **Claude Opus**（強大推理）：技能是否避免過度解釋？

對 Opus 完美有效的內容可能需要對 Haiku 提供更多細節。如果您計劃在多個模型上使用您的技能，請以適合所有模型的指示為目標。

## 技能結構

<Note>
  **YAML 前置資料**：SKILL.md 前置資料支援兩個欄位：

  * `name` — 技能的人類可讀名稱（最多 64 個字元）
  * `description` — 技能功能和使用時機的單行描述（最多 1024 個字元）

  有關完整技能結構詳情，請參閱[技能概述](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名慣例

使用一致的命名模式使技能更易於參考和討論。我們建議使用**動名詞形式**（動詞 + -ing）作為技能名稱，因為這清楚地描述了技能提供的活動或能力。

**好的命名範例（動名詞形式）**：

* "Processing PDFs"
* "Analyzing spreadsheets"
* "Managing databases"
* "Testing code"
* "Writing documentation"

**可接受的替代形式**：

* 名詞短語："PDF Processing"、"Spreadsheet Analysis"
* 行動導向："Process PDFs"、"Analyze Spreadsheets"

**避免**：

* 模糊名稱："Helper"、"Utils"、"Tools"
* 過於通用："Documents"、"Data"、"Files"
* 技能集合中的不一致模式

一致命名使以下事項更容易：

* 在文件和對話中參考技能
* 一眼了解技能的功能
* 組織和搜尋多個技能
* 維護專業、連貫的技能庫

### 撰寫有效的描述

`description` 欄位啟用技能發現，應包含技能的功能和使用時機。

<Warning>
  **始終以第三人稱撰寫**。描述被注入系統提示中，不一致的人稱觀點可能導致發現問題。

  * **好的：** "Processes Excel files and generates reports"
  * **避免：** "I can help you process Excel files"
  * **避免：** "You can use this to process Excel files"
</Warning>

**具體並包含關鍵術語**。包含技能的功能和使用它的具體觸發條件/背景。

每個技能恰好有一個描述欄位。描述對技能選擇至關重要：Claude 使用它從可能 100 個以上可用技能中選擇正確的技能。您的描述必須提供足夠的細節讓 Claude 知道何時選擇此技能，而 SKILL.md 的其餘部分提供實作細節。

有效範例：

**PDF Processing 技能：**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel Analysis 技能：**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git Commit Helper 技能：**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免以下模糊描述：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 漸進式揭露模式

SKILL.md 作為將 Claude 指向所需詳細材料的概述，就像入門指南中的目錄。有關漸進式揭露如何運作的解釋，請參閱概述中的[技能運作方式](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

**實際指引：**

* 保持 SKILL.md 主體在 500 行以下以獲得最佳效能
* 在接近此限制時將內容拆分為獨立檔案
* 使用以下模式有效組織指示、程式碼和資源

#### 視覺概述：從簡單到複雜

基本技能從一個包含元資料和指示的 SKILL.md 檔案開始：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="Simple SKILL.md file showing YAML frontmatter and markdown body" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

隨著技能增長，您可以捆綁 Claude 僅在需要時載入的其他內容：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="Bundling additional reference files like reference.md and forms.md." data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整的技能目錄結構可能如下所示：

```
pdf/
├── SKILL.md              # Main instructions (loaded when triggered)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (executed, not loaded)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

#### 模式 1：帶有參考的高層次指南

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
````

Claude 僅在需要時載入 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### 模式 2：特定領域的組織

對於有多個領域的技能，按領域組織內容以避免載入不相關的上下文。當使用者詢問銷售指標時，Claude 只需要讀取與銷售相關的模式，而非財務或行銷資料。這保持 token 使用率低且上下文聚焦。

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

````markdown SKILL.md theme={null}
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing → See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts → See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption → See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email → See [reference/marketing.md](reference/marketing.md)

## Quick search

Find specific metrics using grep:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### 模式 3：條件式細節

顯示基本內容，連結到進階內容：

```markdown  theme={null}
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

Claude 僅在使用者需要這些功能時才讀取 REDLINING.md 或 OOXML.md。

### 避免深度嵌套參考

當從其他已參考的檔案中參考檔案時，Claude 可能會部分讀取。遇到嵌套參考時，Claude 可能使用 `head -100` 等命令預覽內容而非讀取整個檔案，導致資訊不完整。

**保持參考距 SKILL.md 只有一層深度**。所有參考檔案應直接從 SKILL.md 連結，確保 Claude 在需要時讀取完整檔案。

**壞範例：太深**：

```markdown  theme={null}
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
```

**好範例：一層深度**：

```markdown  theme={null}
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### 為較長的參考檔案建立目錄

對於超過 100 行的參考檔案，在頂部包含目錄。這確保 Claude 即使在部分讀取時也能看到可用資訊的完整範圍。

**範例**：

```markdown  theme={null}
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

Claude 可以讀取完整檔案或根據需要跳至特定章節。

有關此基於檔案系統的架構如何啟用漸進式揭露的詳情，請參閱下方進階章節中的[執行環境](#runtime-environment)。

## 工作流程和回饋循環

### 使用工作流程處理複雜任務

將複雜操作分解為清晰、連續的步驟。對於特別複雜的工作流程，提供 Claude 可以複製到回應中並在進行時勾選的核對清單。

**範例 1：研究綜合工作流程**（適用於不含程式碼的技能）：

````markdown  theme={null}
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
````

此範例展示工作流程如何應用於不需要程式碼的分析任務。核對清單模式適用於任何複雜的多步驟流程。

**範例 2：PDF 表單填寫工作流程**（適用於含程式碼的技能）：

````markdown  theme={null}
## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
````

清晰的步驟防止 Claude 跳過關鍵驗證。核對清單幫助 Claude 和您追蹤多步驟工作流程的進度。

### 實作回饋循環

**常見模式**：執行驗證器 → 修復錯誤 → 重複

此模式大幅提升輸出品質。

**範例 1：樣式指南合規**（適用於不含程式碼的技能）：

```markdown  theme={null}
## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
```

這展示了使用參考文件而非腳本的驗證循環模式。「驗證器」是 STYLE\_GUIDE.md，Claude 透過閱讀和比較來執行檢查。

**範例 2：文件編輯流程**（適用於含程式碼的技能）：

```markdown  theme={null}
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

驗證循環及早捕捉錯誤。

## 內容指引

### 避免時間敏感資訊

不要包含會過時的資訊：

**壞範例：時間敏感**（將變得不正確）：

```markdown  theme={null}
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**好範例**（使用「舊模式」章節）：

```markdown  theme={null}
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

舊模式章節提供歷史背景，不會讓主要內容雜亂。

### 使用一致的術語

選擇一個術語並在整個技能中使用它：

**好的 — 一致**：

* 始終使用「API endpoint」
* 始終使用「field」
* 始終使用「extract」

**壞的 — 不一致**：

* 混用「API endpoint」、「URL」、「API route」、「path」
* 混用「field」、「box」、「element」、「control」
* 混用「extract」、「pull」、「get」、「retrieve」

一致性幫助 Claude 理解和遵循指示。

## 常見模式

### 範本模式

為輸出格式提供範本。根據您的需求匹配嚴格程度。

**嚴格要求**（如 API 回應或資料格式）：

````markdown  theme={null}
## Report structure

ALWAYS use this exact template structure:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
````

**靈活指引**（當調整有用時）：

````markdown  theme={null}
## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive summary
[Overview]

## Key findings
[Adapt sections based on what you discover]

## Recommendations
[Tailor to the specific context]
```

Adjust sections as needed for the specific analysis type.
````

### 範例模式

對於輸出品質取決於看到範例的技能，提供輸入/輸出對，就像在一般提示中一樣：

````markdown  theme={null}
## Commit message format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

Follow this style: type(scope): brief description, then detailed explanation.
````

範例幫助 Claude 比單獨的描述更清楚地理解所需的風格和細節程度。

### 條件式工作流程模式

引導 Claude 完成決策點：

```markdown  theme={null}
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

<Tip>
  如果工作流程因步驟過多而變得龐大複雜，考慮將它們推入獨立檔案，並告訴 Claude 根據手頭的任務讀取適當的檔案。
</Tip>

## 評估和迭代

### 先建立評估

**在撰寫大量文件之前先建立評估。** 這確保您的技能解決真實問題，而非記錄假想的問題。

**評估驅動開發：**

1. **識別差距**：在沒有技能的情況下讓 Claude 執行有代表性的任務。記錄具體的失敗或缺失的上下文
2. **建立評估**：建立測試這些差距的三個場景
3. **建立基準線**：測量沒有技能時 Claude 的效能
4. **撰寫最小指示**：建立剛好足夠解決差距並通過評估的內容
5. **迭代**：執行評估，與基準線比較，並精煉

此方法確保您解決實際問題，而非預測可能永遠不會出現的需求。

**評估結構**：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  此範例展示帶有簡單測試標準的資料驅動評估。我們目前不提供執行這些評估的內建方式。使用者可以建立自己的評估系統。評估是衡量技能有效性的真相來源。
</Note>

### 與 Claude 迭代開發技能

最有效的技能開發過程涉及 Claude 本身。與 Claude 的一個實例（「Claude A」）合作建立將被其他實例（「Claude B」）使用的技能。Claude A 幫助您設計和精煉指示，而 Claude B 在真實任務中測試它們。這有效是因為 Claude 模型理解如何撰寫有效的代理指示以及代理需要什麼資訊。

**建立新技能：**

1. **在沒有技能的情況下完成任務**：與 Claude A 透過正常提示解決問題。在工作過程中，您自然會提供上下文、解釋偏好並分享程序知識。注意您反复提供的資訊。

2. **識別可重複使用的模式**：完成任務後，識別您提供的哪些上下文對未來類似任務有用。

   **範例**：如果您完成了 BigQuery 分析，您可能提供了資料表名稱、欄位定義、過濾規則（如「始終排除測試帳戶」）和常見查詢模式。

3. **請 Claude A 建立技能**：「建立一個捕捉我們剛剛使用的 BigQuery 分析模式的技能。包含資料表模式、命名慣例以及關於過濾測試帳戶的規則。」

   <Tip>
     Claude 模型原生理解技能格式和結構。您不需要特殊系統提示或「writing skills」技能來讓 Claude 幫助建立技能。只需請 Claude 建立技能，它將生成帶有適當前置資料和主體內容的正確格式 SKILL.md 內容。
   </Tip>

4. **審查簡潔性**：檢查 Claude A 是否添加了不必要的解釋。詢問：「刪除關於勝率含義的解釋 — Claude 已經知道。」

5. **改善資訊架構**：請 Claude A 更有效地組織內容。例如：「將此組織為資料表模式在獨立參考檔案中。我們以後可能會添加更多資料表。」

6. **在類似任務上測試**：使用技能與 Claude B（載入技能的新鮮實例）在相關使用案例上。觀察 Claude B 是否找到正確資訊、正確應用規則並成功處理任務。

7. **根據觀察迭代**：如果 Claude B 遇到困難或遺漏某些內容，帶著具體情況返回 Claude A：「當 Claude 使用此技能時，它忘記按 Q4 日期過濾。我們是否應該添加關於日期過濾模式的章節？」

**迭代現有技能：**

改善技能時相同的階層式模式繼續。您在以下兩者之間交替：

* **與 Claude A 合作**（幫助精煉技能的專家）
* **與 Claude B 測試**（使用技能執行真實工作的代理）
* **觀察 Claude B 的行為**並將洞察帶回 Claude A

1. **在真實工作流程中使用技能**：給 Claude B（載入技能）實際任務，而非測試場景

2. **觀察 Claude B 的行為**：注意它在哪裡遇到困難、成功或做出意外選擇

   **範例觀察**：「當我要求 Claude B 提供地區銷售報告時，它撰寫了查詢但忘記過濾測試帳戶，即使技能提到了此規則。」

3. **返回 Claude A 進行改善**：分享當前的 SKILL.md 並描述您觀察到的。詢問：「我注意到 Claude B 在我要求地區報告時忘記過濾測試帳戶。技能提到了過濾，但可能不夠突出？」

4. **審查 Claude A 的建議**：Claude A 可能建議重新組織使規則更突出，使用更強的語言如「MUST filter」而非「always filter」，或重構工作流程章節。

5. **應用並測試更改**：使用 Claude A 的精煉更新技能，然後在類似請求上再次與 Claude B 測試

6. **根據使用重複**：在遇到新場景時繼續此觀察-精煉-測試循環。每次迭代根據真實代理行為（而非假設）改善技能。

**收集團隊回饋：**

1. 與隊友分享技能並觀察他們的使用
2. 詢問：技能是否在預期時激活？指示是否清楚？缺少什麼？
3. 納入回饋以解決您自己使用模式中的盲點

**為何此方法有效**：Claude A 理解代理需求，您提供領域專業知識，Claude B 透過真實使用揭露差距，迭代精煉根據觀察到的行為（而非假設）改善技能。

### 觀察 Claude 如何導航技能

在迭代技能時，注意 Claude 在實踐中如何實際使用它們。注意：

* **意外的探索路徑**：Claude 是否以您未預期的順序讀取檔案？這可能表示您的結構不如您想像的直觀
* **錯失的連接**：Claude 是否未能跟隨重要檔案的參考？您的連結可能需要更明確或更突出
* **過度依賴某些章節**：如果 Claude 反複讀取同一個檔案，考慮該內容是否應放在主 SKILL.md 中
* **被忽略的內容**：如果 Claude 從不存取捆綁的檔案，它可能是不必要的或在主指示中信號不良

根據這些觀察而非假設進行迭代。技能元資料中的 `name` 和 `description` 特別關鍵。Claude 在決定是否根據當前任務觸發技能時使用這些。確保它們清楚描述技能的功能和應使用的時機。

## 要避免的反模式

### 避免 Windows 風格路徑

始終在檔案路徑中使用正斜線，即使在 Windows 上：

* ✓ **好的**：`scripts/helper.py`、`reference/guide.md`
* ✗ **避免**：`scripts\helper.py`、`reference\guide.md`

Unix 風格路徑適用於所有平台，而 Windows 風格路徑在 Unix 系統上會導致錯誤。

### 避免提供太多選項

除非必要，不要呈現多種方法：

````markdown  theme={null}
**Bad example: Too many choices** (confusing):
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**Good example: Provide a default** (with escape hatch):
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
````

## 進階：帶有可執行程式碼的技能

以下章節專注於包含可執行腳本的技能。如果您的技能僅使用 markdown 指示，請跳至[有效技能的核對清單](#checklist-for-effective-skills)。

### 解決問題，不要推卸責任

為技能撰寫腳本時，處理錯誤條件而非推卸給 Claude。

**好範例：明確處理錯誤**：

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return ''
```

**壞範例：推卸給 Claude**：

```python  theme={null}
def process_file(path):
    # Just fail and let Claude figure it out
    return open(path).read()
```

配置參數也應有理由和記錄，以避免「魔法常數」（Ousterhout 定律）。如果您不知道正確的值，Claude 如何確定它？

**好範例：自記錄**：

```python  theme={null}
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

**壞範例：魔法數字**：

```python  theme={null}
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

### 提供工具腳本

即使 Claude 可以撰寫腳本，預先製作的腳本也有優勢：

**工具腳本的好處**：

* 比生成的程式碼更可靠
* 節省 token（無需在上下文中包含程式碼）
* 節省時間（無需生成程式碼）
* 確保跨使用的一致性

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="Bundling executable scripts alongside instruction files" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上方圖表顯示可執行腳本如何與指示檔案一起運作。指示檔案（forms.md）參考腳本，Claude 可以執行它，無需將其內容載入上下文。

**重要區別**：在您的指示中清楚說明 Claude 是否應該：

* **執行腳本**（最常見）：「Run `analyze_form.py` to extract fields」
* **作為參考讀取**（用於複雜邏輯）：「See `analyze_form.py` for the field extraction algorithm」

對於大多數工具腳本，執行是首選，因為它更可靠且高效。請參閱下方[執行環境](#runtime-environment)章節了解腳本執行如何運作的詳情。

**範例**：

````markdown  theme={null}
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or lists conflicts
```

**fill_form.py**: Apply field values to PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### 使用視覺分析

當輸入可以渲染為圖片時，讓 Claude 分析它們：

````markdown  theme={null}
## Form layout analysis

1. Convert PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. Claude can see field locations and types visually
````

<Note>
  在此範例中，您需要撰寫 `pdf_to_images.py` 腳本。
</Note>

Claude 的視覺能力有助於理解佈局和結構。

### 建立可驗證的中間輸出

當 Claude 執行複雜的開放式任務時，可能會犯錯。「計畫-驗證-執行」模式透過讓 Claude 首先以結構化格式建立計畫，然後在執行之前用腳本驗證該計畫，及早發現錯誤。

**範例**：想象要求 Claude 根據電子表格更新 PDF 中的 50 個表單欄位。若無驗證，Claude 可能參考不存在的欄位、建立衝突的值、遺漏必要欄位或錯誤應用更新。

**解決方案**：使用上述工作流程模式（PDF 表單填寫），但在應用更改之前添加一個中間 `changes.json` 檔案進行驗證。工作流程變為：分析 → **建立計畫檔案** → **驗證計畫** → 執行 → 驗證。

**為何此模式有效：**

* **及早發現錯誤**：驗證在應用更改之前發現問題
* **機器可驗證**：腳本提供客觀驗證
* **可逆規劃**：Claude 可以在不觸及原始檔案的情況下迭代計畫
* **清晰除錯**：錯誤訊息指向具體問題

**使用時機**：批次操作、破壞性更改、複雜驗證規則、高風險操作。

**實作提示**：讓驗證腳本提供具體錯誤訊息，如「Field 'signature\_date' not found. Available fields: customer\_name, order\_total, signature\_date\_signed」，幫助 Claude 修復問題。

### 封裝依賴項

技能在具有平台特定限制的程式碼執行環境中執行：

* **claude.ai**：可以從 npm 和 PyPI 安裝套件，並從 GitHub 存儲庫拉取
* **Anthropic API**：沒有網路存取，且沒有執行時套件安裝

在您的 SKILL.md 中列出所需套件，並在[程式碼執行工具文件](/en/docs/agents-and-tools/tool-use/code-execution-tool)中驗證它們是否可用。

### 執行環境

技能在具有檔案系統存取、bash 命令和程式碼執行能力的程式碼執行環境中執行。有關此架構的概念解釋，請參閱概述中的[技能架構](/en/docs/agents-and-tools/agent-skills/overview#the-skills-architecture)。

**這如何影響您的撰寫：**

**Claude 如何存取技能：**

1. **元資料預先載入**：啟動時，所有技能 YAML 前置資料中的名稱和描述被載入系統提示
2. **檔案按需讀取**：Claude 使用 bash Read 工具從檔案系統存取 SKILL.md 和其他檔案
3. **腳本高效執行**：工具腳本可以透過 bash 執行，無需將其完整內容載入上下文。只有腳本的輸出消耗 token
4. **大型檔案無上下文懲罰**：參考檔案、資料或文件在實際讀取之前不消耗上下文 token

* **檔案路徑很重要**：Claude 像檔案系統一樣導航您的技能目錄。使用正斜線（`reference/guide.md`），不要使用反斜線
* **描述性命名檔案**：使用表示內容的名稱：`form_validation_rules.md`，而非 `doc2.md`
* **為發現組織**：按領域或功能組織目錄
  * 好的：`reference/finance.md`、`reference/sales.md`
  * 壞的：`docs/file1.md`、`docs/file2.md`
* **捆綁全面資源**：包含完整 API 文件、豐富範例、大型資料集；在存取之前無上下文懲罰
* **對確定性操作偏好腳本**：撰寫 `validate_form.py` 而非要求 Claude 生成驗證程式碼
* **明確執行意圖**：
  * 「Run `analyze_form.py` to extract fields」（執行）
  * 「See `analyze_form.py` for the extraction algorithm」（作為參考讀取）
* **測試檔案存取模式**：透過真實請求測試，驗證 Claude 可以導航您的目錄結構

**範例：**

```
bigquery-skill/
├── SKILL.md (overview, points to reference files)
└── reference/
    ├── finance.md (revenue metrics)
    ├── sales.md (pipeline data)
    └── product.md (usage analytics)
```

當使用者詢問收入時，Claude 讀取 SKILL.md，看到 `reference/finance.md` 的參考，並呼叫 bash 僅讀取該檔案。sales.md 和 product.md 檔案保留在檔案系統上，消耗零上下文 token，直到需要時。此基於檔案系統的模型使漸進式揭露成為可能。Claude 可以導航並有選擇地載入每個任務所需的確切內容。

有關技術架構的完整詳情，請參閱技能概述中的[技能運作方式](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

### MCP 工具參考

如果您的技能使用 MCP（Model Context Protocol）工具，請始終使用完整限定的工具名稱以避免「工具未找到」錯誤。

**格式**：`ServerName:tool_name`

**範例**：

```markdown  theme={null}
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中：

* `BigQuery` 和 `GitHub` 是 MCP 伺服器名稱
* `bigquery_schema` 和 `create_issue` 是這些伺服器中的工具名稱

沒有伺服器前綴，Claude 可能無法定位工具，特別是當有多個 MCP 伺服器可用時。

### 避免假設工具已安裝

不要假設套件可用：

````markdown  theme={null}
**Bad example: Assumes installation**:
"Use the pdf library to process the file."

**Good example: Explicit about dependencies**:
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## 技術注意事項

### YAML 前置資料要求

SKILL.md 前置資料僅包含 `name`（最多 64 個字元）和 `description`（最多 1024 個字元）欄位。有關完整結構詳情，請參閱[技能概述](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。

### Token 預算

保持 SKILL.md 主體在 500 行以下以獲得最佳效能。如果您的內容超過此限制，使用前面描述的漸進式揭露模式將其拆分為獨立檔案。有關架構詳情，請參閱[技能概述](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

## 有效技能的核對清單

在分享技能之前，驗證：

### 核心品質

* [ ] 描述具體並包含關鍵術語
* [ ] 描述包含技能的功能和使用時機
* [ ] SKILL.md 主體在 500 行以下
* [ ] 其他細節在獨立檔案中（如果需要）
* [ ] 沒有時間敏感資訊（或在「舊模式」章節中）
* [ ] 整個文件使用一致的術語
* [ ] 範例具體，而非抽象
* [ ] 檔案參考只有一層深度
* [ ] 適當使用漸進式揭露
* [ ] 工作流程有清晰步驟

### 程式碼和腳本

* [ ] 腳本解決問題而非推卸給 Claude
* [ ] 錯誤處理明確且有幫助
* [ ] 沒有「魔法常數」（所有值都有理由）
* [ ] 指示中列出了必要套件並已驗證其可用性
* [ ] 腳本有清晰文件
* [ ] 沒有 Windows 風格路徑（所有正斜線）
* [ ] 關鍵操作有驗證/確認步驟
* [ ] 品質關鍵任務包含回饋循環

### 測試

* [ ] 至少建立了三個評估
* [ ] 使用 Haiku、Sonnet 和 Opus 測試
* [ ] 使用真實使用場景測試
* [ ] 納入了團隊回饋（如果適用）

## 後續步驟

<CardGroup cols={2}>
  <Card title="開始使用 Agent Skills" icon="rocket" href="/en/docs/agents-and-tools/agent-skills/quickstart">
    建立您的第一個技能
  </Card>

  <Card title="在 Claude Code 中使用技能" icon="terminal" href="/en/docs/claude-code/skills">
    在 Claude Code 中建立和管理技能
  </Card>

  <Card title="透過 API 使用技能" icon="code" href="/en/api/skills-guide">
    以程式方式上傳和使用技能
  </Card>
</CardGroup>
