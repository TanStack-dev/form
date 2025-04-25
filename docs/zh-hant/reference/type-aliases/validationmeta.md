---
source-updated-at: '2025-04-25T12:58:29.000Z'
translation-updated-at: '2025-04-25T20:53:32.639Z'
id: ValidationMeta
title: 類型 / ValidationMeta
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 類型別名：ValidationMeta

```ts
type ValidationMeta = object;
```

定義於：[packages/form-core/src/FormApi.ts:452](https://github.com/TanStack/form/blob/main/packages/form-core/src/FormApi.ts#L452)

表示欄位驗證元資料的物件，不建議公開使用。

## 類型宣告

### lastAbortController

```ts
lastAbortController: AbortController;
```

儲存在記憶體中的中止控制器 (AbortController)，用於取消先前的非同步驗證嘗試。
