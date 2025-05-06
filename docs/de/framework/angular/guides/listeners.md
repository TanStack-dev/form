---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:22:57.606Z'
id: listeners
title: Side effects for event triggers
---

Für Situationen, in denen Sie auf Trigger "einwirken" oder "reagieren" möchten, gibt es die Listener-API (Listener API). Beispielsweise, wenn Sie als Entwickler ein Formularfeld zurücksetzen möchten, nachdem sich ein anderes Feld geändert hat, würden Sie die Listener-API verwenden.

Stellen Sie sich folgenden Benutzerablauf vor:

- Der Benutzer wählt ein Land aus einer Dropdown-Liste aus.
- Der Benutzer wählt dann eine Provinz aus einer anderen Dropdown-Liste aus.
- Der Benutzer ändert das ausgewählte Land zu einem anderen.

In diesem Beispiel muss, wenn der Benutzer das Land ändert, die ausgewählte Provinz zurückgesetzt werden, da sie nicht mehr gültig ist. Mit der Listener-API können wir uns auf das `onChange`-Event abonnieren und ein Reset für das Feld "province" auslösen, wenn der Listener aktiviert wird.

Events, auf die "gelauscht" werden kann, sind:

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
