---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:19:26.713Z'
id: listeners
title: Side effects for event triggers
---

トリガーに対して「影響を与える」または「反応する」必要がある場合、リスナー API (listener API) を使用できます。例えば、開発者が他のフィールドの変更に応じてフォームフィールドをリセットしたい場合、リスナー API を利用します。

以下のユーザーフローを想像してください:

- ユーザーがドロップダウンから国を選択
- ユーザーが別のドロップダウンから州/省を選択
- ユーザーが選択した国を変更

この例では、ユーザーが国を変更した際、選択された州/省は無効になるためリセットする必要があります。リスナー API を使用すると、`onChange` イベントを購読し、リスナーが発火した時に「province」フィールドにリセットをディスパッチできます。

「リスニング」可能なイベントは以下の通りです:

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
