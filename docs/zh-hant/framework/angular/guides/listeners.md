---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:18:14.950Z'
id: listeners
title: Side effects for event triggers
---

## 事件觸發的副作用 (Side effects)

當您需要「影響」或「回應」觸發事件時，可以使用監聽器 API (listener API)。舉例來說，如果您作為開發者，想要在某個欄位變更時重置另一個表單欄位，就可以使用監聽器 API。

想像以下使用者流程：

- 使用者從下拉選單中選擇國家
- 使用者接著從另一個下拉選單中選擇省份
- 使用者將選定的國家更改為另一個國家

在這個範例中，當使用者變更國家時，已選定的省份需要被重置，因為它不再有效。透過監聽器 API，我們可以訂閱 onChange 事件，並在監聽器觸發時對「省份」欄位發送重置指令。

可被「監聽」的事件包括：

- onChange
- onBlur
- onMount
- onSubmit

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
