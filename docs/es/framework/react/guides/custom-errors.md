---
source-updated-at: '2025-03-04T14:23:57.000Z'
translation-updated-at: '2025-04-30T21:48:40.206Z'
id: custom-errors
title: Errores personalizados
---

# Errores Personalizados

TanStack Form ofrece completa flexibilidad en los tipos de valores de error que puede devolver desde los validadores. Los errores de tipo string son los más comunes y fáciles de trabajar, pero la biblioteca le permite devolver cualquier tipo de valor desde sus validadores.

Como regla general, cualquier valor truthy se considera como un error y marcará el formulario o campo como inválido, mientras que los valores falsy (`false`, `undefined`, `null`, etc.) significan que no hay error, el formulario o campo es válido.

## Devolver valores String desde formularios

```tsx
<form.Field
  name="username"
  validators={{
    onChange: ({ value }) =>
      value.length < 3 ? 'Username must be at least 3 characters' : undefined,
  }}
/>
```

Para validación a nivel de formulario que afecta múltiples campos:

```tsx
const form = useForm({
  defaultValues: {
    username: '',
    email: '',
  },
  validators: {
    onChange: ({ value }) => {
      return {
        fields: {
          username:
            value.username.length < 3 ? 'Username too short' : undefined,
          email: !value.email.includes('@') ? 'Invalid email' : undefined,
        },
      }
    },
  },
})
```

Los errores de tipo string son los más comunes y se muestran fácilmente en su interfaz de usuario:

```tsx
{
  field.state.meta.errors.map((error, i) => (
    <div key={i} className="error">
      {error}
    </div>
  ))
}
```

### Números

Útiles para representar cantidades, umbrales o magnitudes:

```tsx
<form.Field
  name="age"
  validators={{
    onChange: ({ value }) => (value < 18 ? 18 - value : undefined),
  }}
/>
```

Mostrar en la interfaz:

```tsx
{
  /* TypeScript sabe que el error es un número basado en su validador */
}
;<div className="error">
  Necesita {field.state.meta.errors[0]} años más para ser elegible
</div>
```

### Booleanos

Banderas simples para indicar estado de error:

```tsx
<form.Field
  name="accepted"
  validators={{
    onChange: ({ value }) => (!value ? true : undefined),
  }}
/>
```

Mostrar en la interfaz:

```tsx
{
  field.state.meta.errors[0] === true && (
    <div className="error">Debe aceptar los términos</div>
  )
}
```

### Objetos

Objetos de error enriquecidos con múltiples propiedades:

```tsx
<form.Field
  name="email"
  validators={{
    onChange: ({ value }) => {
      if (!value.includes('@')) {
        return {
          message: 'Formato de email inválido',
          severity: 'error',
          code: 1001,
        }
      }
      return undefined
    },
  }}
/>
```

Mostrar en la interfaz:

```tsx
{
  typeof field.state.meta.errors[0] === 'object' && (
    <div className={`error ${field.state.meta.errors[0].severity}`}>
      {field.state.meta.errors[0].message}
      <small> (Código: {field.state.meta.errors[0].code})</small>
    </div>
  )
}
```

En el ejemplo anterior depende del evento de error que desee mostrar.

### Arrays

Múltiples mensajes de error para un solo campo:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      const errors = []
      if (value.length < 8) errors.push('Contraseña demasiado corta')
      if (!/[A-Z]/.test(value)) errors.push('Falta letra mayúscula')
      if (!/[0-9]/.test(value)) errors.push('Falta número')

      return errors.length ? errors : undefined
    },
  }}
/>
```

Mostrar en la interfaz:

```tsx
{
  Array.isArray(field.state.meta.errors) && (
    <ul className="error-list">
      {field.state.meta.errors.map((err, i) => (
        <li key={i}>{err}</li>
      ))}
    </ul>
  )
}
```

## La propiedad `disableErrorFlat` en campos

Por defecto, TanStack Form aplana los errores de todas las fuentes de validación (onChange, onBlur, onSubmit) en un solo array `errors`. La propiedad `disableErrorFlat` preserva las fuentes de error:

```tsx
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }) =>
      !value.includes('@') ? 'Formato de email inválido' : undefined,
    onBlur: ({ value }) =>
      !value.endsWith('.com') ? 'Solo se permiten dominios .com' : undefined,
    onSubmit: ({ value }) =>
      value.length < 5 ? 'Email demasiado corto' : undefined,
  }}
/>
```

Sin `disableErrorFlat`, todos los errores se combinarían en `field.state.meta.errors`. Con esta propiedad, puede acceder a los errores por su fuente:

```tsx
{
  field.state.meta.errorMap.onChange && (
    <div className="real-time-error">{field.state.meta.errorMap.onChange}</div>
  )
}

{
  field.state.meta.errorMap.onBlur && (
    <div className="blur-feedback">{field.state.meta.errorMap.onBlur}</div>
  )
}

{
  field.state.meta.errorMap.onSubmit && (
    <div className="submit-error">{field.state.meta.errorMap.onSubmit}</div>
  )
}
```

Esto es útil para:

- Mostrar diferentes tipos de errores con diferentes tratamientos de UI
- Priorizar errores (ej. mostrar errores de envío más prominentemente)
- Implementar revelación progresiva de errores

## Seguridad de tipos de `errors` y `errorMap`

TanStack Form proporciona fuerte seguridad de tipos para el manejo de errores. Cada clave en `errorMap` tiene exactamente el tipo devuelto por su validador correspondiente, mientras que el array `errors` contiene un tipo unión de todos los posibles valores de error de todos los validadores:

```tsx
<form.Field
  name="password"
  validators={{
    onChange: ({ value }) => {
      // Esto devuelve un string o undefined
      return value.length < 8 ? 'Demasiado corta' : undefined
    },
    onBlur: ({ value }) => {
      // Esto devuelve un objeto o undefined
      if (!/[A-Z]/.test(value)) {
        return { message: 'Falta mayúscula', level: 'warning' }
      }
      return undefined
    },
  }}
  children={(field) => {
    // TypeScript sabe que errors[0] puede ser string | { message: string, level: string } | undefined
    const error = field.state.meta.errors[0]

    // Manejo de errores con seguridad de tipos
    if (typeof error === 'string') {
      return <div className="string-error">{error}</div>
    } else if (error && typeof error === 'object') {
      return <div className={error.level}>{error.message}</div>
    }

    return null
  }}
/>
```

La propiedad `errorMap` también está completamente tipada, coincidiendo con los tipos de retorno de sus funciones de validación:

```tsx
// Con disableErrorFlat
<form.Field
  name="email"
  disableErrorFlat
  validators={{
    onChange: ({ value }): string | undefined =>
      !value.includes("@") ? "Email inválido" : undefined,
    onBlur: ({ value }): { code: number, message: string } | undefined =>
      !value.endsWith(".com") ? { code: 100, message: "Dominio incorrecto" } : undefined
  }}
  children={(field) => {
    // TypeScript conoce el tipo exacto de cada fuente de error
    const onChangeError: string | undefined = field.state.meta.errorMap.onChange;
    const onBlurError: { code: number, message: string } | undefined = field.state.meta.errorMap.onBlur;

    return (/* ... */);
  }}
/>
```

Esta seguridad de tipos ayuda a detectar errores en tiempo de compilación en lugar de en tiempo de ejecución, haciendo su código más confiable y mantenible.
