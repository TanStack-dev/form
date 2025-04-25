---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:11:14.180Z'
id: arrays
title: 数组
---
TanStack Form 支持将数组作为表单值使用，包括数组内的子对象值。

## 基本用法

要使用数组值，可以在数组值上结合 `field.state.value` 和 [`solid-js` 的 `Index`](https://www.solidjs.com/tutorial/flow_index) 实现：

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
          {/* 不要将此替换为 `For`，否则功能将不符合预期 */}
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

> 必须使用 `solid-js` 的 `Index` 而非 `For`，因为 `For` 会导致内部组件在数组变化时重新渲染，
> 这将造成字段值丢失，从而删除子字段的值。

每次对 `field` 调用 `pushValue` 时，都会生成映射后的 JSX：

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  添加人员
</button>
```

子字段可以这样使用：

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

## 完整示例

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
                {/* 不要替换为 For，否则测试将失败 */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>人员 {i} 的姓名</div>
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
                添加人员
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
