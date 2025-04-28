---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:36:39.479Z'
id: react-native
title: React Native
---

## 與 React Native 的搭配使用

TanStack Form 採用無頭設計 (headless)，應該能夠直接支援 React Native 而無需任何額外設定。

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
      <Text>Age:</Text>
      <TextInput value={field.state.value} onChangeText={field.handleChange} />
      {field.state.meta.errors ? (
        <Text>{field.state.meta.errors.join(', ')}</Text>
      ) : null}
    </>
  )}
</form.Field>
```
