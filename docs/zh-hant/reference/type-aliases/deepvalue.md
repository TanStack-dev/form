---
source-updated-at: '2025-04-19T08:22:21.000Z'
translation-updated-at: '2025-04-25T20:52:04.937Z'
id: DeepValue
title: 類型 / DeepValue
---
<!-- 請勿編輯：此頁面由類型註解自動生成 -->

# 類型別名：DeepValue\<TValue, TAccessor\>

```ts
type DeepValue<TValue, TAccessor> = unknown extends TValue ? TValue : TAccessor extends DeepKeys<TValue> ? DeepRecord<TValue>[TAccessor] : never;
```

定義於：[packages/form-core/src/util-types.ts:164](https://github.com/TanStack/form/blob/main/packages/form-core/src/util-types.ts#L164)

推斷物件或陣列中深層嵌套屬性的類型。

## 類型參數

• **TValue**

• **TAccessor**
