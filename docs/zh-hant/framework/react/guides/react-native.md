---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:28:08.865Z'
id: react-native
title: React Native
---

## 動機

多數網頁框架並未提供完整的表單處理解決方案，迫使開發者必須自行實作客製化方案或依賴功能有限的函式庫。這通常會導致一致性不足、效能低落以及開發時間增加等問題。TanStack Form 旨在透過提供一個強大且易用的全方位表單管理解決方案，來應對這些挑戰。

使用 TanStack Form，開發者能解決以下常見的表單相關挑戰：

- 響應式資料綁定與狀態管理
- 複雜的驗證與錯誤處理
- 無障礙性與響應式設計
- 國際化與在地化
- 跨平台相容性與客製化樣式

透過為這些挑戰提供完整解決方案，TanStack Form 讓開發者能輕鬆建構穩固且使用者友善的表單。

TanStack Form 採用無頭式 (headless) 設計，應該能直接支援 React Native 而無需任何額外設定。

以下是一個範例：

```tsx
<form.Field
  name="age"
  validators={{
    onChange: (val) =>
      val < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <Text>年齡：</Text>
      <TextInput value={field.state.value} onChangeText={field.handleChange} />
      {!field.state.meta.isValid && (
        <Text>{field.state.meta.errors.join(', ')}</Text>
      )}
    </>
  )}
</form.Field>
```
