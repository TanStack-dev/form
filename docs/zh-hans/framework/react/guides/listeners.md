---
source-updated-at: '2025-04-25T12:57:21.000Z'
translation-updated-at: '2025-04-29T23:14:33.629Z'
id: listeners
title: 监听器
---

## 事件触发器的副作用处理

当您需要"影响"或"响应"某些触发行为时，可以使用监听器 (listener) API。例如，如果您作为开发者希望在某个字段变化时重置另一个表单字段，就可以使用监听器 API。

考虑以下用户流程：

- 用户从下拉菜单中选择一个国家
- 用户从另一个下拉菜单中选择一个省份
- 用户将所选国家更改为另一个国家

在这个例子中，当用户更改国家时，所选省份需要被重置，因为它不再有效。通过监听器 API，我们可以订阅 `onChange` 事件，并在监听器触发时对"province"字段执行重置操作。

可被"监听"的事件包括：

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

### 内置防抖功能

如果您需要在监听器内发起 API 请求，可能需要对调用进行防抖 (debounce) 处理以避免性能问题。我们提供了简单的方法来为监听器添加防抖功能，只需设置 `onChangeDebounceMs` 或 `onBlurDebounceMs` 即可。

```tsx
<form.Field
  name="country"
  listeners={{
    onChangeDebounceMs: 500, // 500ms 防抖
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

### 表单级监听器

在更高层级上，监听器也可以在表单级别使用，使您可以访问 `onMount` 和 `onSubmit` 事件，并将 `onChange` 和 `onBlur` 事件传播到所有子字段。

`onMount` 和 `onSubmit` 监听器可访问以下属性：

- `formApi`

`onChange` 和 `onBlur` 监听器可访问：

- `fieldApi`
- `formApi`

```tsx
const form = useForm({
  listeners: {
    onMount: ({ formApi }) => {
      // 自定义日志服务
      loggingService('mount', formApi.state.values)
    },

    onChange: ({ formApi, fieldApi }) => {
      // 自动保存逻辑
      if (formApi.state.isValid) {
        formApi.handleSubmit()
      }

      // fieldApi 代表触发事件的字段
      console.log(fieldApi.name, fieldApi.state.value)
    },
  },
})
```
