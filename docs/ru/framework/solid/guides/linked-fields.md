---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:01:25.762Z'
id: linked-fields
title: Связанные поля
---

Вы можете столкнуться с необходимостью связать два поля формы, когда одно из них должно валидироваться при изменении значения другого поля.  
Один из таких случаев — когда у вас есть поля `password` и `confirm_password`, и вы хотите, чтобы `confirm_password` выдавал ошибку, если его значение не совпадает с `password`, независимо от того, какое поле вызвало изменение.

Представьте следующий сценарий:

- Пользователь изменяет поле подтверждения пароля.
- Пользователь изменяет основное поле пароля.

В этом примере форма всё равно будет содержать ошибки, так как валидация поля "подтверждение пароля" не была перезапущена, чтобы отметить его как корректное.

Чтобы решить эту проблему, нужно убедиться, что валидация "подтверждения пароля" перезапускается при обновлении поля пароля.  
Для этого можно добавить свойство `onChangeListenTo` к полю `confirm_password`.

```tsx
export default function App() {
  const form = createForm(() => ({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  }))

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field().state.value}
              onChange={(e) => field().handleChange(e.target.value)}
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
                value={field().state.value}
                onChange={(e) => field().handleChange(e.target.value)}
              />
            </label>
            <Index each={field().state.meta.errors}>
              {(err) => <div>{err()}</div>}
            </Index>
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

Аналогично работает свойство `onBlurListenTo`, которое перезапускает валидацию при потере фокуса полем.
