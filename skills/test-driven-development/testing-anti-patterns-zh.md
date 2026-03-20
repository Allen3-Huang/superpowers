# 測試反模式

**載入此參考資料的時機：** 撰寫或修改測試、添加 mock，或打算向生產程式碼添加僅供測試的方法時。

## 概述

測試必須驗證真實行為，而非 mock 行為。Mock 是用來隔離的手段，不是被測試的對象。

**核心原則：** 測試程式碼的行為，而非 mock 的行為。

**嚴格遵循 TDD 可以防止這些反模式。**

## 鐵律

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## 反模式一：測試 Mock 行為

**違規情況：**
```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**為什麼這是錯的：**
- 你在驗證 mock 是否存在，而非元件是否正常運作
- 當 mock 存在時測試通過，不存在時失敗
- 對真實行為毫無說明

**你的人類夥伴的糾正：** 「我們在測試 mock 的行為嗎？」

**修正方式：**
```typescript
// ✅ GOOD: Test real component or don't mock it
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// OR if sidebar must be mocked for isolation:
// Don't assert on the mock - test Page's behavior with sidebar present
```

### 關卡函式

```
BEFORE asserting on any mock element:
  Ask: "Am I testing real component behavior or just mock existence?"

  IF testing mock existence:
    STOP - Delete the assertion or unmock the component

  Test real behavior instead
```

## 反模式二：生產程式碼中的僅供測試方法

**違規情況：**
```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Looks like production API!
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// In tests
afterEach(() => session.destroy());
```

**為什麼這是錯的：**
- 生產類別被僅供測試的程式碼污染
- 若意外在生產環境呼叫則很危險
- 違反 YAGNI 和關注點分離
- 混淆了物件生命週期與實體生命週期

**修正方式：**
```typescript
// ✅ GOOD: Test utilities handle test cleanup
// Session has no destroy() - it's stateless in production

// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// In tests
afterEach(() => cleanupSession(session));
```

### 關卡函式

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?"

  IF yes:
    STOP - Don't add it
    Put it in test utilities instead

  Ask: "Does this class own this resource's lifecycle?"

  IF no:
    STOP - Wrong class for this method
```

## 反模式三：不理解依賴就進行 Mock

**違規情況：**
```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  // Mock prevents config write that test depends on!
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // Should throw - but won't!
});
```

**為什麼這是錯的：**
- 被 mock 的方法具有測試所依賴的副作用（寫入設定）
- 為了「安全」過度 mock 破壞了實際行為
- 測試因錯誤原因通過，或莫名其妙失敗

**修正方式：**
```typescript
// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  // Mock the slow part, preserve behavior test needs
  vi.mock('MCPServerManager'); // Just mock slow server startup

  await addServer(config);  // Config written
  await addServer(config);  // Duplicate detected ✓
});
```

### 關卡函式

```
BEFORE mocking any method:
  STOP - Don't mock yet

  1. Ask: "What side effects does the real method have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (the actual slow/external operation)
    OR use test doubles that preserve necessary behavior
    NOT the high-level method the test depends on

  IF unsure what test depends on:
    Run test with real implementation FIRST
    Observe what actually needs to happen
    THEN add minimal mocking at the right level

  Red flags:
    - "I'll mock this to be safe"
    - "This might be slow, better mock it"
    - Mocking without understanding the dependency chain
```

## 反模式四：不完整的 Mock

**違規情況：**
```typescript
// ❌ BAD: Partial mock - only fields you think you need
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code uses
};

// Later: breaks when code accesses response.metadata.requestId
```

**為什麼這是錯的：**
- **不完整的 mock 隱藏結構假設** — 你只 mock 了你知道的欄位
- **下游程式碼可能依賴你未包含的欄位** — 靜默失敗
- **測試通過但整合失敗** — Mock 不完整，真實 API 完整
- **虛假的信心** — 測試對真實行為毫無證明

**鐵律：** 按照現實中存在的完整資料結構進行 mock，而非僅 mock 當前測試使用的欄位。

**修正方式：**
```typescript
// ✅ GOOD: Mirror real API completeness
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // All fields real API returns
};
```

### 關卡函式

```
BEFORE creating mock responses:
  Check: "What fields does the real API response contain?"

  Actions:
    1. Examine actual API response from docs/examples
    2. Include ALL fields system might consume downstream
    3. Verify mock matches real response schema completely

  Critical:
    If you're creating a mock, you must understand the ENTIRE structure
    Partial mocks fail silently when code depends on omitted fields

  If uncertain: Include all documented fields
```

## 反模式五：將整合測試視為事後補充

**違規情況：**
```
✅ Implementation complete
❌ No tests written
"Ready for testing"
```

**為什麼這是錯的：**
- 測試是實作的一部分，而非可選的後續步驟
- TDD 會捕捉到這個問題
- 沒有測試就不能聲稱完成

**修正方式：**
```
TDD cycle:
1. Write failing test
2. Implement to pass
3. Refactor
4. THEN claim complete
```

## 當 Mock 變得太複雜時

**警告信號：**
- Mock 設定比測試邏輯更長
- Mock 所有東西才能讓測試通過
- Mock 缺少真實元件擁有的方法
- Mock 改變時測試就壞掉

**你的人類夥伴的問題：** 「我們這裡需要使用 mock 嗎？」

**考慮：** 使用真實元件的整合測試，通常比複雜的 mock 更簡單

## TDD 如何防止這些反模式

**TDD 有幫助的原因：**
1. **先寫測試** → 強迫你思考你實際在測試什麼
2. **看著它失敗** → 確認測試測的是真實行為，而非 mock
3. **最小實作** → 不會悄悄加入僅供測試的方法
4. **真實依賴** → 在 mock 之前你能看到測試實際需要什麼

**如果你在測試 mock 行為，你違反了 TDD** — 你在未看到測試針對真實程式碼失敗的情況下添加了 mock。

## 快速參考

| 反模式 | 修正方式 |
|--------------|-----|
| 在 mock 元素上斷言 | 測試真實元件或取消 mock |
| 生產程式碼中的僅供測試方法 | 移至測試工具程式 |
| 不理解就 mock | 先理解依賴，盡量少 mock |
| 不完整的 mock | 完整映射真實 API |
| 將測試視為事後補充 | TDD——先寫測試 |
| 過於複雜的 mock | 考慮整合測試 |

## 紅旗警示

- 斷言檢查 `*-mock` 測試 ID
- 方法只在測試檔案中被呼叫
- Mock 設定佔測試的 50% 以上
- 移除 mock 時測試失敗
- 無法解釋為什麼需要 mock
- 為了「安全」而 mock

## 結論

**Mock 是用來隔離的工具，不是用來測試的對象。**

如果 TDD 揭示你在測試 mock 行為，你就走錯了方向。

修正方式：測試真實行為，或質疑你為什麼要 mock。
