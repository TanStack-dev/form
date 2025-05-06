---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:27:38.428Z'
id: listeners
title: Side effects for event triggers
---

Для ситуаций, когда необходимо "влиять" или "реагировать" на триггеры, существует API слушателей (listener API). Например, если разработчик хочет сбросить поле формы в результате изменения другого поля, следует использовать API слушателей.

Рассмотрим следующий пользовательский сценарий:

- Пользователь выбирает страну из выпадающего списка.
- Затем пользователь выбирает регион из другого выпадающего списка.
- Пользователь изменяет выбранную страну на другую.

В этом примере при изменении страны выбранный регион необходимо сбросить, так как он становится недействительным. С помощью API слушателей можно подписаться на событие `onChange` и выполнить сброс поля "province" при срабатывании слушателя.

События, на которые можно подписаться:

- `onChange`
- `onBlur`
- `onMount`
- `onSubmit`

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="country"
      [listeners]="{
        onChange: onCountryChange
      }"
      #country="field"
    ></ng-container>

    <ng-container
      [tanstackField]="form"
      name="province"
      #province="field"
    ></ng-container>
  `,
})

export class AppComponent {
  form = injectForm({
    defaultValues: {
      country: '',
      province: '',
    },
  })

  onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`Country changed to: ${value}, resetting province`)
    this.form.setFieldValue('province', '')
  }
}
```
