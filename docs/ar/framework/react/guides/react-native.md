---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:05:06.245Z'
id: react-native
title: React Native
---

# الاستخدام مع React Native (التطبيق الأصلي للتفاعل)

TanStack Form هو بدون واجهة جاهزة (headless) ويجب أن يدعم React Native بشكل جاهز دون الحاجة إلى أي إعدادات إضافية.

إليك مثال:

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
      {!field.state.meta.isValid && (
        <Text>{field.state.meta.errors.join(', ')}</Text>
      )}
    </>
  )}
</form.Field>
```
