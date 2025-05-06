---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:21:09.191Z'
id: listeners
title: Side effects for event triggers
---

Para situaciones donde desee "afectar" o "reaccionar" a activadores, existe la API de escucha (listener API). Por ejemplo, si usted, como desarrollador, desea restablecer un campo del formulario como resultado de que otro campo cambie, utilizaría la API de escucha (listener API).

Imagine el siguiente flujo de usuario:

- El usuario selecciona un país de un menú desplegable.
- Luego, el usuario selecciona una provincia de otro menú desplegable.
- El usuario cambia el país seleccionado por otro diferente.

En este ejemplo, cuando el usuario cambia el país, la provincia seleccionada debe restablecerse porque ya no es válida. Con la API de escucha (listener API), podemos suscribirnos al evento onChange y despachar un restablecimiento al campo "province" cuando se active el escucha.

Los eventos que se pueden "escuchar" son:

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
