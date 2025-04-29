---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-29T23:14:30.211Z'
id: philosophy
title: 设计理念
---

每个成熟的项目都应具备指导其发展的核心理念。缺乏核心理念的开发会陷入无休止的决策困境，最终导致脆弱的 API 设计。

本文档阐述了驱动 TanStack Form 开发和功能设计的核心原则。

## 统一 API 的渐进升级

API 设计必然伴随权衡取舍。虽然为每种取舍方案提供独立 API 看似诱人，但这会导致 API 碎片化，增加学习和使用成本。

尽管统一 API 可能带来更高的学习曲线，但它意味着开发者无需纠结内部该选用哪个 API，也避免了在切换 API 时的认知负担。

## 表单需要灵活性

TanStack Form 被设计为灵活可定制的解决方案。虽然多数表单遵循相似模式，但总存在例外情况——尤其是当表单成为应用核心组件时。

为此，TanStack Form 支持多种验证方式：

- **时机定制**：可在失去焦点时、值变更时、提交时甚至挂载时触发验证
- **验证策略**：支持字段级验证、表单整体验证或字段子集验证
- **自定义验证逻辑**：可编写自有验证逻辑或集成 [Zod](https://zod.dev/) / [Valibot](https://valibot.dev/) 等验证库
- **自定义错误信息**：通过验证器返回任意对象来定制每个字段的错误提示
- **异步验证**：支持异步字段验证，并内置防抖和取消等常用工具

## 受控即王道

在受控与非受控输入组件的争论中，TanStack Form 坚定选择受控方案。

这带来诸多优势：

- **可预测性**：可随时预判表单状态
- **易测试性**：通过传入值并断言输出即可轻松测试表单
- **非 DOM 支持**：兼容 React Native、Three.js 框架适配器及其他渲染器
- **增强的条件逻辑**：可根据表单状态轻松控制字段显隐
- **调试友好**：可直接将表单状态输出至控制台进行调试

## 泛型的局限性

使用 TanStack Form 时，开发者无需手动传递泛型参数或使用内部类型。我们通过运行时默认值实现了完整的类型推断。

编写符合规范的 TanStack Form 代码时，除了对运行时值的类型断言外，JavaScript 与 TypeScript 的使用体验应该毫无差异。

应避免：

```typescript
useForm<MyForm>()
```

而应采用：

```typescript
interface Person {
  name: string
  age: number
}

const defaultPerson: Person = { name: 'Bill Luo', age: 24 }

useForm({
  defaultValues: defaultPerson,
})
```

## 封装即解放

TanStack Form 的核心目标之一，是让开发者能将其封装进自有组件系统或设计系统中。

为此我们提供了诸多工具来简化自定义组件和钩子的构建：

```typescript
// 从自有库导出已绑定表单组件的预配置钩子
export const { useAppForm, withForm } = createFormHook(/* options */)
```

若不进行此类封装，应用中将出现大量样板代码，导致表单不一致且降低用户体验。
