---
source-updated-at: '2025-04-25T12:57:21.000Z'
translation-updated-at: '2025-04-25T20:33:46.836Z'
id: listeners
title: 監聽器
---
## 事件觸發的副作用 (Side effects)

當您需要「影響」或「回應」觸發事件時，可以使用監聽器 API (listener API)。舉例來說，如果您作為開發者，想要在某個欄位變更時重置另一個表單欄位，就可以使用監聽器 API。

想像以下使用者流程：

- 使用者從下拉選單中選擇國家
- 接著從另一個下拉選單中選擇省份
- 使用者又將選定的國家更改為其他國家

在這個範例中，當使用者變更國家時，已選定的省份需要被重置，因為它不再有效。透過監聽器 API，我們可以訂閱 `onChange` 事件，並在監聽器觸發時對「省份」欄位發送重置指令。

可監聽的事件包括：

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      country: '',
      province: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field
        name="country"
        listeners={{
          onChange: ({ value }) => {
            console.log(`Country changed to: ${value}, resetting province`)
            form.setFieldValue('province', '')
          },
        }}
      >
        {(field) => (
          <label>
            <div>Country</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>

      <form.Field name="province">
        {(field) => (
          <label>
            <div>Province</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
    </div>
  )
}
```

### 內建防抖動 (Debouncing)

如果您需要在監聽器內發送 API 請求，可能會希望對呼叫進行防抖動處理，以避免效能問題。我們提供簡易的方法來實現監聽器防抖動，只需添加 `onChangeDebounceMs` 或 `onBlurDebounceMs` 參數即可。

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // 500ms 防抖動
    onChange: ({ value }) => {
      console.log(`Country changed to: ${value} without a change within 500ms, resetting province`)
      form.setFieldValue('province', '')
    },
  }}
>
  {(field) => (
    /* ... */
  )}
</form.Field>
```

### 表單層級監聽器

在更高層級的表單層面也提供監聽器功能，讓您可以存取 `onMount` 和 `onSubmit` 事件，並將 `onChange` 和 `onBlur` 事件傳播給所有子欄位。

`onMount` 和 `onSubmit` 監聽器可存取以下屬性：

- `formApi`

`onChange` 和 `onBlur` 監聽器則可存取：

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // 自訂紀錄服務
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // 自動儲存邏輯
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApi 代表觸發事件的欄位
      console.log(fieldApi.name, fieldApi.state.value)
    },
  },
})
```
