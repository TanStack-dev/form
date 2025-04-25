---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:08:53.544Z'
id: arrays
title: 数组
---
TanStack Form 支持将数组作为表单值，包括数组内的子对象值。

## 基础用法

要使用数组，可以对数组值使用 `field.state.value`：

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

每次在 `field` 上调用 `pushValue` 时都会生成映射后的 JSX：

```jsx
<button onClick={() => field.pushValue({ name: '', age: 0 })} type="button">
  添加人员
</button>
```

最后，可以像这样使用子字段：

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

## 完整示例

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
                              <div>人员 {i} 的姓名</div>
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
                  添加人员
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
