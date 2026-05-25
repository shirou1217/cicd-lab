# CI Pipeline Report

- 學號：`110081014`
- Workflow 檔名：`.github/workflows/ci_110081014.yaml`

## CI Pipeline 說明

本專案使用 GitHub Actions 建立一條在 `push` 事件時自動觸發的 CI pipeline。流程會在 Ubuntu runner 上執行，先安裝 Node.js 與專案依賴，再依序進行以下三項檢查：

1. `npm run typecheck`
2. `npm run format:check`
3. `npm run test:ci`

只要其中任一個步驟失敗，GitHub Actions job 就會立即標示為失敗，因此能確保型別、程式格式與測試結果都通過後才算 CI 成功。

## `.github/workflows/ci_110081014.yaml` 主要內容

```yaml
name: CI

on:
  push:
    branches:
      - '**'

permissions:
  contents: read
  checks: write
  pull-requests: write

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/setup-node@v5
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run typecheck
      - run: npm run format:check
      - run: npm run test:ci
      - if: always()
        uses: dorny/test-reporter@v2
        with:
          name: Vitest Results
          path: test-results/vitest.junit.xml
          reporter: java-junit
```

## Pipeline 設計說明

### 1. 觸發條件

- 使用 `on.push`，只要有程式碼 push 到 GitHub 就會自動執行。

### 2. Node.js 環境

- 使用 `actions/setup-node@v5` 安裝 Node.js 22。
- 啟用 `cache: npm`，降低後續安裝時間。

### 3. 程式品質檢查

- `typecheck`：透過 TypeScript 編譯器確認型別正確。
- `format:check`：透過 Prettier 驗證程式格式是否符合規範。
- `test:ci`：透過 Vitest 執行測試。

### 4. 測試結果直接顯示在 GitHub Actions 頁面

- `test:ci` 會讓 Vitest 除了在 console 顯示測試結果，也額外輸出 `test-results/vitest.junit.xml`。
- workflow 使用 `dorny/test-reporter@v2` 讀取 JUnit XML，將測試摘要與失敗明細直接呈現在 GitHub Actions / Checks 結果頁面。
- 這種做法不需要另外下載 artifact 才能查看測試結果，符合題目要求。

### 5. 失敗策略

- GitHub Actions 預設只要 `run` step 回傳非 0 exit code 就會讓 job 失敗。
- 因此：
  - TypeScript 型別錯誤會使 `npm run typecheck` 失敗
  - Prettier 格式錯誤會使 `npm run format:check` 失敗
  - 測試失敗會使 `npm run test:ci` 失敗

## 使用工具與策略

- GitHub Actions：建立 CI pipeline
- `actions/checkout@v5`：取出 repository 程式碼
- `actions/setup-node@v5`：設定 Node.js 執行環境
- TypeScript：型別檢查
- Prettier：程式格式檢查
- Vitest：執行單元測試
- `dorny/test-reporter@v2`：將 JUnit 測試結果直接顯示於 GitHub Actions 結果頁

策略上採用「先做快速失敗的靜態檢查，再跑測試」的順序。這樣若型別或格式本身就有問題，可以更早終止 workflow，避免浪費 runner 時間。

## CI 執行結果截圖

### 成功案例

請在 push 成功後，插入至少一張 GitHub Actions 成功頁面的截圖：

- 建議截圖內容：
  - workflow 名稱 `CI`
  - `TypeScript typecheck`、`Prettier check`、`Run tests with JUnit output` 都成功
  - `Vitest Results` 測試摘要顯示於頁面

可在這裡貼圖：

`[請貼上成功執行截圖]`

### GitHub Actions 結果頁面

請補上一張 GitHub Actions / Checks 頁面中，能直接看到測試結果摘要的畫面。

可在這裡貼圖：

`[請貼上結果頁面截圖]`

## 失敗案例說明

本次示範可以故意製造一個 `Prettier` 格式錯誤，因為最容易控制且不會影響功能邏輯。

### 製造錯誤方式

例如故意把某個 TypeScript 檔案改成不符合 Prettier 規範的格式，然後 push 到 GitHub。

### 預期結果

- `npm run format:check` 失敗
- GitHub Actions workflow 顯示紅色 failed
- 其他後續步驟不會被視為成功完成

### 錯誤原因

Prettier 會檢查專案內檔案是否符合 `.prettierrc` 設定。若程式碼縮排、引號、逗號或換行格式不符合規範，`prettier --check .` 會回傳非 0 exit code，導致 pipeline 失敗。

### 修正方式

在本機執行：

```bash
npm run format
```

修正格式後再次 commit / push，pipeline 即可恢復成功。

### 失敗截圖

請補上一張 GitHub Actions failed 畫面截圖，並標示失敗 step 為 `Prettier check`。

可在這裡貼圖：

`[請貼上 failed 截圖]`

## 補充

目前專案中已新增 `package.json` 的 `test:ci` script，讓 Vitest 在 CI 中輸出 JUnit XML 檔案，供 `dorny/test-reporter` 直接解析並顯示結果。
