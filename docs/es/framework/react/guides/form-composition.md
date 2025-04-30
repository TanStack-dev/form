---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:49:21.965Z'
id: form-composition
title: Composición de formularios
---

# Composición de Formularios

Una crítica común a TanStack Form es su verbosidad por defecto. Si bien esto _puede_ ser útil con fines educativos —ayudando a reforzar la comprensión de nuestras APIs— no es ideal para casos de uso en producción.

Como resultado, aunque `form.Field` permite el uso más potente y flexible de TanStack Form, proporcionamos APIs que lo envuelven y hacen que el código de tu aplicación sea menos verboso.

## Hooks de Formulario Personalizados

La forma más potente de componer formularios es crear hooks de formulario personalizados. Esto te permite crear un hook de formulario adaptado a las necesidades de tu aplicación, incluyendo componentes de UI pre-vinculados y más.

En su forma más básica, `createFormHook` es una función que toma un `fieldContext` y un `formContext` y devuelve un hook `useAppForm`.

> Este hook `useAppForm` sin personalizar es idéntico a `useForm`, pero eso cambiará rápidamente a medida que agreguemos más opciones a `createFormHook`.

```tsx
import { createFormHookContexts, createFormHook } from '@tanstack/react-form'

// exporta useFieldContext para usar en tus componentes personalizados
export const { fieldContext, formContext, useFieldContext } =
  createFormHookContexts()

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  // Aprenderemos más sobre estas opciones más adelante
  fieldComponents: {},
  formComponents: {},
})

function App() {
  const form = useAppForm({
    // Soporta todas las opciones de useForm
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <form.Field /> // ...
}
```

### Componentes de Campo Pre-Vinculados

Una vez que este andamiaje esté en su lugar, puedes empezar a agregar componentes de campo y formulario personalizados a tu hook de formulario.

> Nota: el `useFieldContext` debe ser el mismo que se exporta desde tu contexto de formulario personalizado

```tsx
import { useFieldContext } from './form-context.tsx'

export function TextField({ label }: { label: string }) {
  // El `Field` infiere que debe tener un tipo `value` de `string`
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

Luego puedes registrar este componente con tu hook de formulario.

```tsx
import { TextField } from './text-field.tsx'

const { useAppForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

Y usarlo en tu formulario:

```tsx
function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    // Observa el `AppField` en lugar de `Field`; `AppField` proporciona el contexto requerido
    <form.AppField
      name="firstName"
      children={(field) => <field.TextField label="First Name" />}
    />
  )
}
```

Esto no solo te permite reutilizar la UI de tu componente compartido, sino que también conserva la seguridad de tipos que esperarías de TanStack Form: un error tipográfico en `name` generará un error de TypeScript.

#### Una nota sobre rendimiento

Aunque el contexto es una herramienta valiosa en el ecosistema de React, muchos usuarios expresan preocupación sobre si proporcionar un valor reactivo a través de un contexto causará re-renderizados innecesarios.

> ¿No estás familiarizado con esta preocupación de rendimiento? [El artículo de Mark Erikson explicando por qué Redux resuelve muchos de estos problemas](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/) es un buen punto de partida.

Aunque es una preocupación válida, no es un problema para TanStack Form; los valores proporcionados a través del contexto no son reactivos en sí mismos, sino que son instancias de clase estáticas con propiedades reactivas ([usando TanStack Store como nuestra implementación de señales para impulsar el funcionamiento](https://tanstack.com/store)).

### Componentes de Formulario Pre-Vinculados

Mientras que `form.AppField` resuelve muchos de los problemas con el boilerplate de Field y la reutilización, no resuelve el problema del boilerplate y la reutilización del _formulario_.

En particular, poder compartir instancias de `form.Subscribe` para, por ejemplo, un botón de envío reactivo es un caso de uso común.

```tsx
function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {},
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return (
    <form.AppForm>
      // Observa el componente envoltorio `AppForm`; `AppForm` proporciona el
      contexto requerido
      <form.SubscribeButton label="Submit" />
    </form.AppForm>
  )
}
```

## Dividir formularios grandes en partes más pequeñas

A veces los formularios se vuelven muy grandes; simplemente sucede. Aunque TanStack Form soporta bien formularios grandes, nunca es divertido trabajar con archivos de cientos o miles de líneas de código.

Para solucionar esto, soportamos dividir formularios en partes más pequeñas usando el componente de orden superior `withForm`.

```tsx
const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

const ChildForm = withForm({
  // Estos valores solo se usan para verificación de tipos, y no se usan en tiempo de ejecución
  // Esto te permite usar `...formOpts` desde `formOptions` sin necesidad de redeclarar las opciones
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
  // Opcional, pero agrega props a la función `render` además de `form`
  props: {
    // Estas props también se establecen como valores predeterminados para la función `render`
    title: 'Child Form',
  },
  render: function Render({ form, title }) {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

function App() {
  const form = useAppForm({
    defaultValues: {
      firstName: 'John',
      lastName: 'Doe',
    },
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

### Preguntas Frecuentes sobre `withForm`

> ¿Por qué un componente de orden superior en lugar de un hook?

Aunque los hooks son el futuro de React, los componentes de orden superior siguen siendo una herramienta poderosa para la composición. En particular, la API de `useForm` nos permite tener una fuerte seguridad de tipos sin requerir que los usuarios pasen genéricos.

> ¿Por qué estoy recibiendo errores de ESLint sobre hooks en `render`?

ESLint busca hooks en el nivel superior de una función, y `render` puede no ser reconocido como un componente de nivel superior, dependiendo de cómo lo hayas definido.

```tsx
// Esto causará errores de ESLint con el uso de hooks
const ChildForm = withForm({
  // ...
  render: ({ form, title }) => {
    // ...
  },
})
```

```tsx
// Esto funciona bien
const ChildForm = withForm({
  // ...
  render: function Render({ form, title }) {
    // ...
  },
})
```

## Tree-shaking de componentes de formulario y campo

Aunque los ejemplos anteriores son excelentes para comenzar, no son ideales para ciertos casos de uso donde podrías tener cientos de componentes de formulario y campo.
En particular, es posible que no desees incluir todos tus componentes de formulario y campo en el bundle de cada archivo que use tu hook de formulario.

Para solucionar esto, puedes combinar la API `createFormHook` de TanStack con los componentes `lazy` y `Suspense` de React:

```typescript
// src/hooks/form-context.ts
import { createFormHookContexts } from '@tanstack/react-form'

export const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()
```

```tsx
// src/components/text-field.tsx
import { useFieldContext } from '../hooks/form-context.tsx'

export default function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()

  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}
```

```tsx
// src/hooks/form.ts
import { lazy } from 'react'
import { createFormHook } from '@tanstack/react-form'

const TextField = lazy(() => import('../components/text-fields.tsx'))

const { useAppForm, withForm } = createFormHook({
  fieldContext,
  formContext,
  fieldComponents: {
    TextField,
  },
  formComponents: {},
})
```

```tsx
// src/App.tsx
import { Suspense } from 'react'
import { PeoplePage } from './features/people/page.tsx'

export default function App() {
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <PeopleForm />
    </Suspense>
  )
}
```

Esto mostrará el fallback de Suspense mientras se carga el componente `TextField`, y luego renderizará el formulario una vez cargado.

## Uniendo todo

Ahora que hemos cubierto los conceptos básicos de creación de hooks de formulario personalizados, unamos todo en un solo ejemplo.

```tsx
// /src/hooks/form.ts, para usar en toda la aplicación
const { fieldContext, useFieldContext, formContext, useFormContext } =
  createFormHookContexts()

function TextField({ label }: { label: string }) {
  const field = useFieldContext<string>()
  return (
    <label>
      <div>{label}</div>
      <input
        value={field.state.value}
        onChange={(e) => field.handleChange(e.target.value)}
      />
    </label>
  )
}

function SubscribeButton({ label }: { label: string }) {
  const form = useFormContext()
  return (
    <form.Subscribe selector={(state) => state.isSubmitting}>
      {(isSubmitting) => <button disabled={isSubmitting}>{label}</button>}
    </form.Subscribe>
  )
}

const { useAppForm, withForm } = createFormHook({
  fieldComponents: {
    TextField,
  },
  formComponents: {
    SubscribeButton,
  },
  fieldContext,
  formContext,
})

// /src/features/people/shared-form.ts, para usar en las características de `people`
const formOpts = formOptions({
  defaultValues: {
    firstName: 'John',
    lastName: 'Doe',
  },
})

// /src/features/people/nested-form.ts, para usar en la página de `people`
const ChildForm = withForm({
  ...formOpts,
  // Opcional, pero agrega props a la función `render` fuera de `form`
  props: {
    title: 'Child Form',
  },
  render: ({ form, title }) => {
    return (
      <div>
        <p>{title}</p>
        <form.AppField
          name="firstName"
          children={(field) => <field.TextField label="First Name" />}
        />
        <form.AppForm>
          <form.SubscribeButton label="Submit" />
        </form.AppForm>
      </div>
    )
  },
})

// /src/features/people/page.ts
const Parent = () => {
  const form = useAppForm({
    ...formOpts,
  })

  return <ChildForm form={form} title={'Testing'} />
}
```

## Guía de Uso de la API

Aquí hay un gráfico para ayudarte a decidir qué APIs deberías usar:

![](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/react_form_composability.svg)
