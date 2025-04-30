---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:48:18.218Z'
id: ui-libraries
title: Bibliotecas de UI
---

## Uso de TanStack Form con bibliotecas de interfaz de usuario (UI Libraries)

TanStack Form es una biblioteca sin interfaz predefinida (headless), lo que le ofrece total flexibilidad para estilizarla según sus preferencias. Es compatible con una amplia gama de bibliotecas de interfaz de usuario, incluyendo `Tailwind`, `Material UI`, `Mantine` o incluso CSS plano.

Esta guía se centra en `Material UI` y `Mantine`, pero los conceptos son aplicables a cualquier biblioteca de interfaz de usuario que elija.

### Requisitos previos

Antes de integrar TanStack Form con una biblioteca de interfaz de usuario, asegúrese de tener instaladas las dependencias necesarias en su proyecto:

- Para `Material UI`, siga las instrucciones de instalación en su [sitio oficial](https://mui.com/material-ui/getting-started/).
- Para `Mantine`, consulte su [documentación](https://mantine.dev/).

Nota: Aunque puede combinar bibliotecas, generalmente se recomienda utilizar solo una para mantener la consistencia y minimizar el código redundante.

### Ejemplo con Mantine

Aquí tiene un ejemplo que demuestra la integración de TanStack Form con componentes de Mantine:

```tsx
import { TextInput, Checkbox } from '@mantine/core'
import { useForm } from '@tanstack/react-form'

export default function App() {
  const { Field, handleSubmit, state } = useForm({
    defaultValues: {
      firstName: '',
      lastName: '',
      isChecked: false,
    },
    onSubmit: async ({ value }) => {
      // Handle form submission
      console.log(value)
    },
  })

  return (
    <>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          handleSubmit()
        }}
      >
        <Field
          name="firstName"
          children={({ state, handleChange, handleBlur }) => (
            <TextInput
              defaultValue={state.value}
              onChange={(e) => handleChange(e.target.value)}
              onBlur={handleBlur}
              placeholder="Enter your name"
            />
          )}
        />
        <Field
          name="isChecked"
          children={({ state, handleChange, handleBlur }) => (
            <Checkbox
              onChange={(e) => handleChange(e.target.checked)}
              onBlur={handleBlur}
              checked={state.value}
            />
          )}
        />
      </form>
      <div>
        <pre>{JSON.stringify(state.values, null, 2)}</pre>
      </div>
    </>
  )
}
```

- Inicialmente, utilizamos el hook `useForm` de TanStack y desestructuramos las propiedades necesarias. Este paso es opcional; alternativamente, podría usar `const form = useForm()` si lo prefiere. La inferencia de tipos de TypeScript garantiza una experiencia fluida independientemente del enfoque.
- El componente `Field`, derivado de `useForm`, acepta varias propiedades, como `validators`. Para esta demostración, nos centramos en dos propiedades principales: `name` y `children`.
  - La propiedad `name` identifica cada `Field`, por ejemplo, `firstName` en nuestro ejemplo.
  - La propiedad `children` aprovecha el concepto de render props, permitiéndonos integrar componentes sin abstracciones innecesarias.
- El diseño de TanStack depende en gran medida de render props, proporcionando acceso a `children` dentro del componente `Field`. Este enfoque es completamente seguro en cuanto a tipos. Al integrar con componentes de Mantine, como `TextInput`, desestructuramos selectivamente propiedades como `state.value`, `handleChange` y `handleBlur`. Este enfoque selectivo se debe a las ligeras diferencias en los tipos entre `TextInput` y el `field` que obtenemos en los children.
- Siguiendo estos pasos, puede integrar sin problemas los componentes de Mantine con TanStack Form.
- Esta metodología es igualmente aplicable a otros componentes, como `Checkbox`, garantizando una integración consistente en diferentes elementos de interfaz de usuario.

### Uso con Material UI

El proceso para integrar componentes de Material UI es similar. Aquí tiene un ejemplo usando TextField y Checkbox de Material UI:

```tsx
        <Field
            name="lastName"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <TextField
                  id="filled-basic"
                  label="Filled"
                  variant="filled"
                  defaultValue={state.value}
                  onChange={(e) => handleChange(e.target.value)}
                  onBlur={handleBlur}
                  placeholder="Enter your last name"
                />
              );
            }}
          />

           <Field
            name="isMuiCheckBox"
            children={({ state, handleChange, handleBlur }) => {
              return (
                <MuiCheckbox
                  onChange={(e) => handleChange(e.target.checked)}
                  onBlur={handleBlur}
                  checked={state.value}
                />
              );
            }}
          />

```

- El enfoque de integración es el mismo que con Mantine.
- La principal diferencia radica en las propiedades específicas y las opciones de estilización de los componentes de Material UI.
