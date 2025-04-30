---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:50:09.680Z'
id: basic-concepts
title: Conceptos básicos
---

# Conceptos Básicos y Terminología

Esta página introduce los conceptos básicos y la terminología utilizada en la biblioteca `@tanstack/lit-form`. Familiarizarse con estos conceptos le ayudará a comprender y trabajar mejor con la biblioteca y su uso con Lit.

## Opciones del Formulario (Form Options)

Puede crear opciones para su formulario para que puedan ser compartidas entre múltiples formularios utilizando la función `formOptions`.

Por ejemplo:

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

## Instancia del Formulario (Form Instance)

Una Instancia del Formulario es un objeto que representa un formulario individual y proporciona métodos y propiedades para trabajar con el formulario. Usted crea una instancia del formulario utilizando la interfaz `TanStackFormController` proporcionada por `@tanstack/lit-form`. El `TanStackFormController` se instancia con la clase del formulario actual (`this`) y algunas opciones predeterminadas del formulario. Inicializa el estado del formulario, maneja el envío del formulario y proporciona métodos para gestionar los campos del formulario y su validación.

```tsx
#form = new TanStackFormController(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

También puede crear una instancia del formulario sin usar `formOptions` utilizando la API independiente de `TanStackFormController`:

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## Campo (Field)

Un Campo representa un único elemento de entrada del formulario, como un campo de texto o una casilla de verificación. Los campos se crean utilizando el método `field(FieldOptions, callback)` proporcionado por la instancia del formulario. El componente acepta un objeto `FieldOptions` y una función de callback que recibe un objeto `FieldApi`. Este objeto proporciona métodos para obtener el valor actual del campo, manejar cambios en la entrada y manejar eventos de desenfoque (blur).

Por ejemplo:

```ts
 ${this.#form.field(
    {
      name: `firstName`,
      validators: {
        onChange: ({ value }) =>
          value.length < 3 ? "Not long enough" : undefined,
        },
      },
      (field: FieldApi<Employee, "firstName">) => {
        return html` <div>
          <label class="first-name-label">First Name</label>
          <input
           id="firstName"
           type="text"
           class="first-name-input"
           placeholder="First Name"
           @blur="${() => field.handleBlur()}"
           .value="${field.state.value}"
           @input="${(event: InputEvent) => {
           if (event.currentTarget) {
            const newValue = (event.currentTarget as HTMLInputElement).value;
            field.handleChange(newValue);
           }
          }}"
        />
      </div>`;
    },
)}
```

## Estado del Campo (Field State)

Cada campo tiene su propio estado, que incluye su valor actual, estado de validación, mensajes de error y otros metadatos. Puede acceder al estado de un campo utilizando su propiedad `field.state`.

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Hay tres estados del campo que pueden ser muy útiles para ver cómo interactúa el usuario con un campo. Un campo está _"tocado" (touched)_ cuando el usuario hace clic o se desplaza hacia él, _"prístino" (pristine)_ hasta que el usuario cambia su valor, y _"modificado" (dirty)_ después de que el valor ha sido cambiado. Puede verificar estos estados mediante las banderas `isTouched`, `isPristine` e `isDirty`, como se muestra a continuación.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Estados del campo](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
