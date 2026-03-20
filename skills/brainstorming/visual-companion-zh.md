# 視覺輔助工具指南

基於瀏覽器的視覺腦力激盪輔助工具，用於展示模型圖、圖表和選項。

## 使用時機

按問題決定，而非按工作階段決定。測試：**使用者透過看到它比閱讀它能更好地理解嗎？**

**使用瀏覽器** 當內容本身是視覺性的：

- **UI 模型圖** — 線框圖、版面、導覽結構、元件設計
- **架構圖** — 系統元件、資料流、關係地圖
- **並排視覺比較** — 比較兩種版面、兩種配色方案、兩種設計方向
- **設計精煉** — 當問題關於外觀與感覺、間距、視覺層次結構時
- **空間關係** — 以圖表呈現的狀態機、流程圖、實體關係

**使用終端機** 當內容是文字或表格式的：

- **需求與範疇問題** — 「X 是什麼意思？」、「哪些功能在範疇內？」
- **概念性 A/B/C 選擇** — 在用文字描述的方案之間選擇
- **取捨清單** — 優缺點、比較表格
- **技術決策** — API 設計、資料建模、架構方案選擇
- **澄清問題** — 任何答案是文字而非視覺偏好的問題

*關於* UI 主題的問題並不自動是視覺問題。「您想要什麼樣的精靈？」是概念性的——使用終端機。「這些精靈版面哪個感覺更合適？」是視覺性的——使用瀏覽器。

## 運作方式

伺服器監視一個目錄中的 HTML 檔案，並將最新的一個提供給瀏覽器。您撰寫 HTML 內容，使用者在其瀏覽器中看到它，並可以點擊選擇選項。選擇結果記錄在您下一輪讀取的 `.events` 檔案中。

**內容片段與完整文件：** 如果您的 HTML 檔案以 `<!DOCTYPE` 或 `<html` 開頭，伺服器將按原樣提供（只注入輔助腳本）。否則，伺服器會自動將您的內容包裝在框架模板中——添加標頭、CSS 主題、選擇指示器以及所有互動基礎設施。**預設撰寫內容片段。** 只有在需要完全控制頁面時，才撰寫完整文件。

## 啟動工作階段

```bash
# 啟動伺服器並持久化（模型圖儲存至專案）
scripts/start-server.sh --project-dir /path/to/project

# 回傳：{"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000"}
```

從回應中儲存 `screen_dir`。告訴使用者開啟 URL。

**尋找連線資訊：** 伺服器將其啟動 JSON 寫入 `$SCREEN_DIR/.server-info`。如果您在背景啟動伺服器且未擷取標準輸出，請讀取該檔案以取得 URL 和連接埠。使用 `--project-dir` 時，請在 `<project>/.superpowers/brainstorm/` 中查找工作階段目錄。

**注意：** 傳入專案根目錄作為 `--project-dir`，使模型圖持久化在 `.superpowers/brainstorm/` 中並在伺服器重新啟動後存活。若無此參數，檔案會存至 `/tmp` 並被清除。如果 `.superpowers/` 尚未加入 `.gitignore`，請提醒使用者添加。

**依平台啟動伺服器：**

**Claude Code（macOS / Linux）：**
```bash
# 預設模式有效——腳本自身將伺服器放入背景
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code（Windows）：**
```bash
# Windows 自動偵測並使用前景模式，這會阻塞工具呼叫。
# 在 Bash 工具呼叫上使用 run_in_background: true，使伺服器在
# 對話輪次之間存活。
scripts/start-server.sh --project-dir /path/to/project
```
透過 Bash 工具呼叫時，設定 `run_in_background: true`。然後在下一輪讀取 `$SCREEN_DIR/.server-info` 以取得 URL 和連接埠。

**Codex：**
```bash
# Codex 會回收背景程序。腳本自動偵測 CODEX_CI 並
# 切換至前景模式。正常執行——不需要額外標誌。
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI：**
```bash
# 使用 --foreground 並在您的 shell 工具呼叫上設定 is_background: true
# 使程序在輪次之間存活
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**其他環境：** 伺服器必須在對話輪次之間持續在背景執行。如果您的環境會回收分離的程序，請使用 `--foreground` 並透過您平台的背景執行機制啟動命令。

如果 URL 從您的瀏覽器無法存取（在遠端/容器化設定中常見），請綁定非回環主機：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

使用 `--url-host` 控制在回傳的 URL JSON 中列印的主機名稱。

## 迴圈

1. **確認伺服器正在運行**，然後**將 HTML 寫入** `screen_dir` 中的新檔案：
   - 每次寫入前，確認 `$SCREEN_DIR/.server-info` 存在。如果不存在（或存在 `.server-stopped`），伺服器已關閉——在繼續前使用 `start-server.sh` 重新啟動。伺服器在閒置 30 分鐘後會自動退出。
   - 使用語意化檔案名稱：`platform.html`、`visual-style.html`、`layout.html`
   - **絕不重複使用檔案名稱** — 每個畫面使用全新的檔案
   - 使用 Write 工具 — **絕不使用 cat/heredoc**（會在終端機中傾倒雜訊）
   - 伺服器自動提供最新的檔案

2. **告訴使用者預期看到什麼，並結束您的輪次：**
   - 提醒他們 URL（每個步驟都要，不只是第一次）
   - 提供螢幕上內容的簡短文字摘要（例如，「顯示首頁的 3 種版面選項」）
   - 請他們在終端機中回應：「請查看並告訴我您的想法。如果您想選擇一個選項，可以點擊。」

3. **在您的下一輪** — 使用者在終端機回應後：
   - 如果存在，請讀取 `$SCREEN_DIR/.events`——這包含使用者的瀏覽器互動（點擊、選擇）作為 JSON 行
   - 與使用者的終端機文字合併以獲得完整圖景
   - 終端機訊息是主要回饋；`.events` 提供結構化互動資料

4. **迭代或前進** — 如果回饋改變了當前畫面，請撰寫新檔案（例如，`layout-v2.html`）。只有在當前步驟已驗證後，才移至下一個問題。

5. **返回終端機時卸載** — 當下一步不需要瀏覽器時（例如，澄清問題、取捨討論），推送等待畫面以清除過時內容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   這可防止使用者在對話已繼續時仍盯著已解決的選擇。當下一個視覺問題出現時，照常推送新的內容檔案。

6. 重複直到完成。

## 撰寫內容片段

只撰寫放入頁面中的內容。伺服器自動將其包裝在框架模板中（標頭、主題 CSS、選擇指示器以及所有互動基礎設施）。

**最簡範例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

就這樣。不需要 `<html>`、CSS 或 `<script>` 標籤。伺服器提供所有這些。

## 可用的 CSS 類別

框架模板為您的內容提供以下 CSS 類別：

### 選項（A/B/C 選擇）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**多選：** 在容器上添加 `data-multiselect` 以讓使用者選擇多個選項。每次點擊切換項目。指示器欄顯示計數。

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### 卡片（視覺設計）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### 模型圖容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### 分割視圖（並排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### 優缺點

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### 模擬元素（線框圖建構模組）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 排版與章節

- `h2` — 頁面標題
- `h3` — 章節標題
- `.subtitle` — 標題下方的次要文字
- `.section` — 帶有下邊距的內容區塊
- `.label` — 小型大寫標籤文字

## 瀏覽器事件格式

當使用者在瀏覽器中點擊選項時，其互動記錄在 `$SCREEN_DIR/.events`（每行一個 JSON 物件）。當您推送新畫面時，檔案會自動清除。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整的事件流顯示使用者的探索路徑——他們可能在確定之前點擊多個選項。最後一個 `choice` 事件通常是最終選擇，但點擊模式可能揭示值得詢問的猶豫或偏好。

如果 `.events` 不存在，使用者沒有與瀏覽器互動——僅使用其終端機文字。

## 設計技巧

- **根據問題調整保真度** — 版面問題用線框圖，精煉問題用精緻設計
- **在每頁說明問題** — 「哪種版面感覺更專業？」而不只是「選一個」
- **前進前先迭代** — 如果回饋改變了當前畫面，撰寫新版本
- **每個畫面最多 2-4 個選項**
- **在重要時使用真實內容** — 對於攝影作品集，使用真實圖片（Unsplash）。佔位符內容會遮蔽設計問題。
- **保持模型圖簡單** — 專注於版面和結構，而非像素完美的設計

## 檔案命名

- 使用語意化名稱：`platform.html`、`visual-style.html`、`layout.html`
- 絕不重複使用檔案名稱——每個畫面必須是新檔案
- 對於迭代：附加版本後綴，如 `layout-v2.html`、`layout-v3.html`
- 伺服器依修改時間提供最新的檔案

## 清理

```bash
scripts/stop-server.sh $SCREEN_DIR
```

如果工作階段使用了 `--project-dir`，模型圖檔案會持久化在 `.superpowers/brainstorm/` 中以供後續參考。只有 `/tmp` 工作階段在停止時會被刪除。

## 參考資料

- 框架模板（CSS 參考）：`scripts/frame-template.html`
- 輔助腳本（客戶端）：`scripts/helper.js`
