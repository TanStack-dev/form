---
source-updated-at: '2025-03-20T03:01:05.000Z'
translation-updated-at: '2025-04-25T20:33:36.696Z'
id: reactivity
title: 響應式
---
Tanstack Form 在與表單互動時不會觸發重新渲染 (re-renders)。因此，您可能會發現自己試圖使用表單或欄位狀態值卻無法成功。

如果您想存取響應式 (reactive) 值，您需要使用以下兩種方法之一來訂閱它們：`useStore` 或 `form.Subscribe` 元件。

這些訂閱的用途包括渲染最新的欄位值、根據條件決定渲染內容，或在元件邏輯中使用欄位值。

> 若您想「響應」觸發事件，請參閱 [listener](./listeners.md) API。

## useStore

當您需要在元件邏輯中存取表單值時，`useStore` 鉤子 (hook) 是最佳選擇。`useStore` 接受兩個參數：首先是表單儲存庫 (store)，其次是一個選擇器 (selector)，用於精確訂閱您想關注的表單部分。

```tsx
const firstName = useStore(form.store, (state) => state.values.firstName)
const errors = useStore(form.store, (state) => state.errorMap)
```

您可以在選擇器中存取表單狀態的任何部分。

> 請注意，當訂閱的值變更時，`useStore` 會導致整個元件重新渲染 (re-render)。

雖然可以省略選擇器，但請避免這樣做，因為省略它會導致每當表單狀態有任何變更時，都會觸發許多不必要的重新渲染。

## form.Subscribe

當您需要在元件的使用者介面 (UI) 中對某些內容做出反應時，`form.Subscribe` 元件是最適合的選擇。例如，根據表單欄位的值顯示或隱藏 UI 元素。

```tsx
<form.Subscribe
  selector={(state) => state.values.firstName}
  children={(firstName) => (
    <form.Field>
      {(field) => (
        <input
          name="lastName"
          value={field.state.lastName}
          onChange={field.handleChange}
        />
      )}
    </form.Field>
  )}
/>
```

> `form.Subscribe` 元件不會觸發元件層級的重新渲染。每當訂閱的值變更時，只有 `form.Subscribe` 元件本身會重新渲染。

選擇使用 `useStore` 還是 `form.Subscribe` 主要取決於以下情況：如果需要渲染在 UI 中，請選擇 `form.Subscribe` 以獲得其優化優勢；如果需要在邏輯中實現響應性 (reactivity)，則應選擇 `useStore`。
