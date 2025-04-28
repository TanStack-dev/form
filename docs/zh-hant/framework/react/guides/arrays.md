---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:33:38.845Z'
id: arrays
title: 陣列
---

TanStack Form 支援將陣列 (array) 作為表單值使用，包括陣列中的子物件值。

## 基本用法

要使用陣列，可以在陣列值上使用 `field.state.value`：

```jsx
function App() {
  const form = useForm({
    defaultValues: {
      people: [],
    },
  })

  return (
    <form.Field name="people" mode="array">
      {(field) => (
        <div>
          {field.state.value.map((_, i) => {
            // ...
          })}
        </div>
      )}
    </form.Field>
  )
}
```

每次在 `field` 上執行 `pushValue` 時，都會生成對應的 JSX：

```jsx
<button onClick={() => field.pushValue({ name: '', age: 0 })} type="button">
  新增人員
</button>
```

最後，可以像這樣使用子欄位 (subfield)：

```jsx
<form.Field key={i} name={`people[${i}].name`}>
  {(subField) => (
    <input
      value={subField.state.value}
      onChange={(e) => subField.handleChange(e.target.value)}
    />
  )}
</form.Field>
```

## 完整範例

```jsx
function App() {
  const form = useForm({
    defaultValues: {
      people: [],
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <form.Field name="people" mode="array">
          {(field) => {
            return (
              <div>
                {field.state.value.map((_, i) => {
                  return (
                    <form.Field key={i} name={`people[${i}].name`}>
                      {(subField) => {
                        return (
                          <div>
                            <label>
                              <div>人員 {i} 的姓名</div>
                              <input
                                value={subField.state.value}
                                onChange={(e) =>
                                  subField.handleChange(e.target.value)
                                }
                              />
                            </label>
                          </div>
                        )
                      }}
                    </form.Field>
                  )
                })}
                <button
                  onClick={() => field.pushValue({ name: '', age: 0 })}
                  type="button"
                >
                  新增人員
                </button>
              </div>
            )
          }}
        </form.Field>
        <form.Subscribe
          selector={(state) => [state.canSubmit, state.isSubmitting]}
          children={([canSubmit, isSubmitting]) => (
            <button type="submit" disabled={!canSubmit}>
              {isSubmitting ? '...' : '提交'}
            </button>
          )}
        />
      </form>
    </div>
  )
}
```
