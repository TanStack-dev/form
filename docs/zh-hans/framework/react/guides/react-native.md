---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:22:56.911Z'
id: react-native
title: React Native
---

## 动机

大多数 Web 框架并未提供全面的表单处理方案，迫使开发者要么自行实现定制方案，要么依赖功能有限的库。这通常会导致一致性缺失、性能低下以及开发时间增加。TanStack Form 旨在通过提供一套强大易用的全功能表单管理方案来解决这些问题。

使用 TanStack Form，开发者可以应对以下常见表单挑战：

- 响应式数据绑定与状态管理
- 复杂验证与错误处理
- 可访问性与响应式设计
- 国际化与本地化
- 跨平台兼容性与自定义样式

通过为这些挑战提供完整解决方案，TanStack Form 让开发者能够轻松构建健壮且用户友好的表单。

TanStack Form 是无头（headless）设计，应当开箱即用地支持 React Native，无需额外配置。

以下是一个示例：

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
      <Text>年龄：</Text>
      <TextInput value={field.state.value} onChangeText={field.handleChange} />
      {!field.state.meta.isValid && (
        <Text>{field.state.meta.errors.join(', ')}</Text>
      )}
    </>
  )}
</form.Field>
```
