---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:59:09.505Z'
id: linked-fields
title: Связанные поля
---

Вы можете столкнуться с необходимостью связать два поля формы, когда одно из них должно валидироваться при изменении значения другого поля.  
Один из распространённых примеров — поля `password` и `confirm_password`, где `confirm_password` должно выдавать ошибку, если его значение не совпадает с `password`, независимо от того, какое поле вызвало изменение.

Представьте следующий сценарий:

- Пользователь изменяет поле подтверждения пароля.
- Пользователь изменяет основное поле пароля.

В этом примере форма всё равно будет содержать ошибки, так как валидация поля "confirm password" не была перезапущена, чтобы отметить его как корректное.

Чтобы решить эту проблему, нужно убедиться, что валидация "confirm password" выполняется заново при обновлении поля пароля.  
Для этого можно добавить свойство `onChangeListenTo` для поля `confirm_password`.

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
      <form.Field
        name="confirm_password"
        validators={{
          onChangeListenTo: ['password'],
          onChange: ({ value, fieldApi }) => {
            if (value !== fieldApi.form.getFieldValue('password')) {
              return 'Passwords do not match'
            }
            return undefined
          },
        }}
      >
        {(field) => (
          <div>
            <label>
              <div>Confirm Password</div>
              <input
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.value)}
              />
            </label>
            {field.state.meta.errors.map((err) => (
              <div key={err}>{err}</div>
            ))}
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

Аналогично работает свойство `onBlurListenTo`, которое перезапускает валидацию при потере фокуса полем.
