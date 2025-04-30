---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:49:57.320Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/angular-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca.

## Instancia de Formulario (Form Instance)

Una Instancia de Formulario (Form Instance) es un objeto que representa un formulario individual y proporciona métodos y propiedades para trabajar con él. Usted crea una instancia de formulario utilizando la función `injectForm`. La función acepta un objeto con una función `onSubmit`, que se llama cuando se envía el formulario.

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // Hacer algo con los datos del formulario
    console.log(value)
  },
})
```

## Campo (Field)

Un Campo (Field) representa un elemento individual de entrada en un formulario, como un campo de texto o una casilla de verificación. Los campos se crean utilizando la directiva `tanstackField`. La directiva acepta una propiedad `name`, que debe coincidir con una clave en los valores predeterminados del formulario. También expone una instancia nombrada `field` de los internos de la directiva, que debe usarse mediante una variable de plantilla para acceder a los internos del campo.

Ejemplo:

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## Estado del Campo (Field State)

Cada campo tiene su propio estado, que incluye su valor actual, estado de validación, mensajes de error y otros metadatos. Puede acceder al estado de un campo utilizando la propiedad `fieldApi.state`.

Ejemplo:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Hay tres estados de campo que pueden ser muy útiles para ver cómo interactúa el usuario con un campo. Un campo está _"touched"_ (tocado) cuando el usuario hace clic o se desplaza hacia él, _"pristine"_ (prístino) hasta que el usuario cambia su valor, y _"dirty"_ (modificado) después de que el valor ha sido cambiado. Puede verificar estos estados mediante las banderas `isTouched`, `isPristine` e `isDirty`, como se muestra a continuación.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## API del Campo (Field API)

La API del Campo (Field API) es un objeto al que se accede en la propiedad `tanstackField.api` al crear un campo. Proporciona métodos para trabajar con el estado del campo.

Ejemplo:

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## Validación

`@tanstack/angular-form` proporciona validación tanto síncrona como asíncrona de forma predeterminada. Las funciones de validación se pueden pasar a la directiva `tanstackField` utilizando la propiedad `validators`.

Ejemplo:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="firstName" #firstName="field">
      <input
        [value]="firstName.api.state.value"
        (blur)="firstName.api.handleBlur()"
        (input)="firstName.api.handleChange($any($event).target.value)"
      />
    </ng-container>
  `,
})
export class AppComponent {
  firstNameValidator: FieldValidateFn<any, any, string, any> = ({
                                                                       value,
                                                                     }) =>
    !value
      ? 'Se requiere un nombre'
      : value.length < 3
        ? 'El nombre debe tener al menos 3 caracteres'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && 'No se permite "error" en el nombre'
    }

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      console.log(value)
    },
  })
}
```

## Validación con Bibliotecas de Esquema Estándar

Además de las opciones de validación personalizadas, también se admite la especificación [Standard Schema](https://github.com/standard-schema/standard-schema).

Puede definir un esquema utilizando cualquiera de las bibliotecas que implementen la especificación y pasarlo a un validador de formulario o campo.

Las bibliotecas compatibles incluyen:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

Ejemplo:

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="firstName"
      [validators]="{
        onChange: z.string().min(3, 'El nombre debe tener al menos 3 caracteres'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: firstNameAsyncValidator
      }"
      #firstName="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  firstNameAsyncValidator = z.string().refine(
    async (value) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return !value.includes('error')
    },
    {
      message: "No se permite 'error' en el nombre",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // Hacer algo con los datos del formulario
      console.log(value)
    },
  })

  z = z
}
```

## Reactividad

`@tanstack/angular-form` ofrece una forma de suscribirse a cambios en el estado del formulario y los campos mediante `injectStore(this.form, selector)`.

Ejemplo:

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## Oyentes (Listeners)

`@tanstack/angular-form` le permite reaccionar a desencadenantes específicos y "escucharlos" para despachar efectos secundarios.

Ejemplo:

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
  `,
})

...

onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`País cambiado a: ${value}, reiniciando provincia`)
    this.form.setFieldValue('province', '')
  }
```

Puede encontrar más información en [Oyentes](./listeners.md).

## Campos de Arreglo (Array Fields)

Los campos de arreglo (Array Fields) le permiten gestionar una lista de valores dentro de un formulario, como una lista de pasatiempos. Puede crear un campo de arreglo utilizando la directiva `tanstackField`.

Al trabajar con campos de arreglo, puede usar los métodos `pushValue`, `removeValue` y `swapValues` para agregar, eliminar e intercambiar valores en el arreglo.

Ejemplo:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        Pasatiempos
        <div>
          @if (!hobbies.api.state.value.length) {
            No se encontraron pasatiempos
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">Nombre:</label>
                  <input
                    [id]="hobbyName.api.name"
                    [name]="hobbyName.api.name"
                    [value]="hobbyName.api.state.value"
                    (blur)="hobbyName.api.handleBlur()"
                    (input)="
                      hobbyName.api.handleChange($any($event).target.value)
                    "
                  />
                  <button
                    type="button"
                    (click)="hobbies.api.removeValue($index)"
                  >
                    X
                  </button>
                </div>
              </ng-container>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyDesc($index)"
                #hobbyDesc="field"
              >
                <div>
                  <label [for]="hobbyDesc.api.name">Descripción:</label>
                  <input
                    [id]="hobbyDesc.api.name"
                    [name]="hobbyDesc.api.name"
                    [value]="hobbyDesc.api.state.value"
                    (blur)="hobbyDesc.api.handleBlur()"
                    (input)="
                      hobbyDesc.api.handleChange($any($event).target.value)
                    "
                  />
                </div>
              </ng-container>
            </div>
          }
        </div>
        <button type="button" (click)="hobbies.api.pushValue(defaultHobby)">
          Agregar pasatiempo
        </button>
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  defaultHobby = {
    name: '',
    description: '',
    yearsOfExperience: 0,
  }

  getHobbyName = (idx: number) => `hobbies[${idx}].name` as const;
  getHobbyDesc = (idx: number) => `hobbies[${idx}].description` as const;

  form = injectForm({
    defaultValues: {
      hobbies: [] as Array<{
        name: string
        description: string
        yearsOfExperience: number
      }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

Estos son los conceptos básicos y la terminología utilizados en la biblioteca `@tanstack/angular-form`. Comprender estos conceptos le ayudará a trabajar de manera más efectiva con la biblioteca y crear formularios complejos con facilidad.
