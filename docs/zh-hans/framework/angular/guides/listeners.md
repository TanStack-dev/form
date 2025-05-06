---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:17:25.002Z'
id: listeners
title: Side effects for event triggers
---

## 事件触发器的副作用处理

当您需要"影响"或"响应"某些触发操作时，可以使用监听器 API (listener API)。例如，如果您作为开发者希望在某个字段变化时重置另一个表单字段，就应该使用监听器 API。

考虑以下用户操作流程：

- 用户从下拉菜单中选择一个国家
- 用户从另一个下拉菜单中选择省份
- 用户将已选国家更改为另一个国家

在这个例子中，当用户更改国家时，已选的省份需要被重置，因为它不再有效。通过监听器 API，我们可以订阅 onChange 事件，并在监听器触发时对"省份"字段执行重置操作。

可被"监听"的事件包括：

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
