# 縱深防禦驗證

## 概述

當你修復因無效資料引起的錯誤時，在一個地方添加驗證感覺已經足夠。但那個單一檢查可能被不同的程式碼路徑、重構或模擬繞過。

**核心原則：** 在資料通過的每一層進行驗證。使錯誤在結構上不可能發生。

## 為何需要多層

單一驗證：「我們修復了錯誤」
多層驗證：「我們使錯誤不可能發生」

不同層次捕獲不同情況：
- 入口驗證捕獲大多數錯誤
- 業務邏輯捕獲邊緣情況
- 環境守衛防止特定情境的危險
- 除錯日誌在其他層失敗時提供幫助

## 四個層次

### 第一層：入口點驗證
**目的：** 在 API 邊界拒絕明顯無效的輸入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### 第二層：業務邏輯驗證
**目的：** 確保資料對此操作有意義

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### 第三層：環境守衛
**目的：** 防止在特定情境中執行危險操作

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### 第四層：除錯儀器
**目的：** 擷取上下文以供鑑識

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 應用此模式

當你發現錯誤時：

1. **追蹤資料流** - 無效值從何而來？在哪裡使用？
2. **繪製所有檢查點** - 列出資料通過的每個點
3. **在每一層添加驗證** - 入口、業務、環境、除錯
4. **測試每一層** - 嘗試繞過第一層，驗證第二層是否能捕獲

## 來自工作階段的範例

錯誤：空的 `projectDir` 導致在原始碼中執行 `git init`

**資料流：**
1. 測試設定 → 空字串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 中執行

**新增的四個層次：**
- 第一層：`Project.create()` 驗證非空、存在且可寫入
- 第二層：`WorkspaceManager` 驗證 `projectDir` 非空
- 第三層：`WorktreeManager` 在測試中拒絕在 tmpdir 以外執行 `git init`
- 第四層：在 `git init` 之前記錄堆疊追蹤

**結果：** 所有 1847 個測試通過，錯誤無法重現

## 關鍵洞察

所有四個層次都是必要的。在測試期間，每一層都捕獲了其他層遺漏的錯誤：
- 不同的程式碼路徑繞過了入口驗證
- 模擬繞過了業務邏輯檢查
- 不同平台上的邊緣情況需要環境守衛
- 除錯日誌識別了結構性誤用

**不要只在一個驗證點停下來。** 在每一層添加檢查。
