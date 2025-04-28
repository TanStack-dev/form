---
source-updated-at: '2025-03-01T06:06:46.000Z'
translation-updated-at: '2025-04-25T20:31:16.928Z'
id: typescript
title: TypeScript
---

TanStack Form 完全使用 **TypeScript** 編寫，具備最高品質的泛型 (generics)、約束 (constraints) 和介面 (interfaces)，確保函式庫與您的專案能達到最高程度的型別安全 (type-safe)！

注意事項：

- 您的 `tsconfig.json` 中必須設定 `strict: true` 才能充分發揮 TanStack Form 的型別功能
- 目前型別系統要求使用 TypeScript v5.4 或更高版本
- 本儲存庫中的型別變更視為**非破壞性變更 (non-breaking)**，通常會以**修補版本 (patch)** 的語意化版本 (semver) 形式發布（否則每個型別增強都會變成主版本號更新！）
- **強烈建議您將 react-form 套件版本鎖定在特定的修補版本，並在升級時預期型別可能在任意版本間被修正或升級**
- TanStack Form 的非型別相關公開 API 仍嚴格遵循語意化版本規範。
