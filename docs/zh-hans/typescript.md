---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-12T04:08:19.788Z'
id: typescript
title: TypeScript
---
TanStack Form 完全采用 **TypeScript** 编写，具备最高质量的泛型、约束和接口设计，确保该库与您的项目实现最大程度的类型安全！

注意事项：

- 您的 `tsconfig.json` 中必须启用 `strict: true` 才能充分利用 TanStack Form 的类型系统
- 当前类型系统要求使用 TypeScript v5.4 或更高版本
- 本仓库中的类型变更被视为**非破坏性变更**，通常以 **patch** 版本号发布（否则每个类型增强都会导致主版本号变更！）
- **强烈建议您将 react-form 包版本锁定到特定 patch 版本，并在升级时预期类型可能在任何版本间被修复或升级**
- TanStack Form 与类型无关的公共 API 仍严格遵循语义化版本规范。
