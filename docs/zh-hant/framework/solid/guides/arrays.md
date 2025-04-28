---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:47:44.864Z'
id: arrays
title: 陣列
---

TanStack Form 支援將陣列 (array) 作為表單值，包含陣列中的子物件值。

## 基本用法

要使用陣列，您可以對陣列值使用 `field.state.value`，並搭配 [`solid-js` 的 `Index`](https://www.solidjs.com/tutorial/flow_index)：

```jsx
function App() {
  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
  }))

  return (
    <form.Field name="people" mode="array">
      {(field) => (
        <Show when={field().state.value.length > 0}>
          {/* 請勿將此處改為 `For`，否則將無法如預期運作 */}
          <Index each={field().state.value}>
            {
              (_, i) => null // ...
            }
          </Index>
        </Show>
      )}
    </form.Field>
  )
}
```

> 您必須使用 `solid-js` 的 `Index` 而非 `For`，因為 `For` 會在陣列變更時導致內部元件重新渲染。
>
> 這會造成欄位值遺失，進而刪除子欄位的值。

當您在 `field` 上執行 `pushValue` 時，將會生成映射後的 JSX：

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  新增人員
</button>
```

最後，您可以像這樣使用子欄位：

```jsx
<form.Field name={`people[${i}].name`}>
  {(subField) => (
    <input
      value={subField().state.value}
      onInput={(e) => {
        subField().handleChange(e.currentTarget.value)
      }}
    />
  )}
</form.Field>
```

## 完整範例

```jsx
function App() {
  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
    onSubmit: ({ value }) => alert(JSON.stringify(value)),
  }))

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <form.Field name="people">
          {(field) => (
            <div>
              <Show when={field().state.value.length > 0}>
                {/* 請勿將此處改為 For，否則測試將失敗 */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>人員 {i} 的姓名</div>
                            <input
                              value={subField().state.value}
                              onInput={(e) => {
                                subField().handleChange(e.currentTarget.value)
                              }}
                            />
                          </label>
                        </div>
                      )}
                    </form.Field>
                  )}
                </Index>
              </Show>

              <button
                onClick={() => field().pushValue({ name: '', age: 0 })}
                type="button"
              >
                新增人員
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">提交</button>
      </form>
    </div>
  )
}
```
