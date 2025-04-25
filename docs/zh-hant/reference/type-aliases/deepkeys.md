---
source-updated-at: '2025-04-19T08:22:21.000Z'
translation-updated-at: '2025-04-25T20:52:04.099Z'
id: DeepKeys
title: 類型 / DeepKeys
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 類型別名: DeepKeys\<T\>

```ts
type DeepKeys<T> = unknown extends T ? string : DeepKeysAndValues<T>["key"];
```

定義於: [packages/form-core/src/util-types.ts:157](https://github.com/TanStack/form/blob/main/packages/form-core/src/util-types.ts#L157)

取得物件或陣列的鍵名 (key)，包含深層嵌套的鍵名。

## 類型參數

• **T**
