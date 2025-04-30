---
source-updated-at: '2025-03-01T08:30:20.000Z'
translation-updated-at: '2025-04-30T21:47:13.029Z'
id: quick-start
title: Inicio rápido
---

# Inicio Rápido

El mínimo indispensable para comenzar con TanStack Form es crear un `TanstackFormController` como se muestra a continuación con la interfaz `Employee` para nuestro formulario de prueba:

```ts
interface Employee {
  firstName: string
  lastName: string
  employed: boolean
  jobTitle: string
}

#form = new TanStackFormController()<Employee>(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  },
})
```

En este ejemplo, `this` hace referencia a la instancia de su `LitElement` en la que desea utilizar TanStack Form.

Para conectar un elemento del formulario en su plantilla con TanStack Form, utilice el método `field` de `TanstackFormController`.

El primer parámetro de `field` es `FieldOptions` y el segundo es una función de devolución de llamada para renderizar su elemento.

```ts
field(FieldOptions, callback)
```

Nuestro formulario de prueba completado debería verse como se muestra a continuación. El formulario recoge el nombre del usuario desde un campo de entrada:

```ts
export class TestForm extends LitElement {
  #form = new TanStackFormController<Employee>(this, {
    defaultValues: {
      firstName: '',
      lastName: '',
      employed: false,
      jobTitle: '',
    },
  })
  render() {
    return html` <p>Por favor, ingrese su nombre></p>
      ${this.#form.field(
        {
          name: `firstName`,
          validators: {
            onChange: ({ value }) =>
              value.length < 3 ? 'No es lo suficientemente largo' : undefined,
          },
        },
        (field: FieldApi<Employee, 'firstName'>) => {
          return html` <div>
            <label class="first-name-label">Nombre</label>
            <input
              id="firstName"
              type="text"
              placeholder="Nombre"
              @blur="${() => field.handleBlur()}"
              .value="${field.getValue()}"
              @input="${(event: InputEvent) => {
                if (event.currentTarget) {
                  const newValue = (event.currentTarget as HTMLInputElement)
                    .value
                  field.handleChange(newValue)
                }
              }}"
            />
          </div>`
        },
      )}`
  }
}
```

Tenga en cuenta que necesita manejar la actualización del elemento y el formulario usted mismo, como se muestra en el ejemplo anterior.
