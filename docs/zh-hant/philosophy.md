---
source-updated-at: '2025-04-16T08:45:06.000Z'
translation-updated-at: '2025-04-25T20:31:39.116Z'
id: philosophy
title: 設計理念
---
每個成熟的專案都應該有其指導開發的核心哲學。若缺乏核心理念，開發過程可能會陷入無止境的決策困境，最終導致 API 設計薄弱。

本文概述了驅動 TanStack Form 開發與功能設計的核心原則。

## 升級統一化的 API

API 設計必然伴隨權衡取捨。因此，開發者常會想透過不同 API 來提供各種權衡方案。但這可能導致 API 碎片化，增加學習與使用難度。

雖然這意味著較高的學習曲線，但能讓您無需糾結該選用哪種 API，也不會在切換 API 時增加認知負擔。

## 表單需要靈活性

TanStack Form 的設計強調靈活性與可定制化。雖然多數表單可能遵循相似模式，但總會存在例外情況——尤其是當表單成為應用核心元件時。

因此，TanStack Form 支援多種驗證方式：

- **時機定制**：可在失焦時、變更時、提交時，甚至掛載時進行驗證
- **驗證策略**：可針對單一欄位、整個表單或欄位子集進行驗證
- **自訂驗證邏輯**：可自行編寫驗證邏輯，或使用 [Zod](https://zod.dev/) 或 [Valibot](https://valibot.dev/) 等函式庫
- **自訂錯誤訊息**：可透過驗證器返回任意物件來定制每個欄位的錯誤訊息
- **非同步驗證**：支援非同步欄位驗證，並自動處理防抖與取消等常見工具

## 受控式設計的優勢

在「受控與非受控輸入」的熱門議題中，TanStack Form 堅定採用受控式設計。

這帶來諸多優勢：

- **可預測性**：隨時掌握表單狀態
- **易於測試**：透過傳入值與斷言輸出即可輕鬆測試表單
- **非 DOM 支援**：可搭配 React Native、Three.js 框架轉接器或任何渲染器使用
- **強化條件邏輯**：能根據表單狀態輕鬆顯示/隱藏欄位
- **除錯便利**：可輕鬆將表單狀態輸出至控制台進行除錯

## 避免泛型參數

使用 TanStack Form 時，您不應需要傳遞泛型參數或使用內部型別。這是因為函式庫設計會從運行時預設值推斷所有型別。

當編寫足夠正確的 TanStack Form 程式碼時，除了對運行時值的型別轉換外，您應該無法區分 JavaScript 與 TypeScript 的使用差異。

不建議寫法：
```typescript
useForm<MyForm>()
```

建議寫法：
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

## 函式庫的解放力量

TanStack Form 的主要目標之一，是讓您能將其封裝進自己的元件系統或設計系統中。

為支援此目標，我們提供多種工具來簡化自訂元件與鉤子的開發：

```typescript
// 從您的函式庫匯出已預綁定表單元件的工具
export const { useAppForm, withForm } = createFormHook(/* options */)
```

若不這麼做，將大幅增加應用程式的樣板程式碼，並降低表單的一致性和使用者體驗。
