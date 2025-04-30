---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:47:27.343Z'
id: quick-start
title: Inicio rápido
---

## Motivación

La mayoría de los frameworks web no ofrecen una solución integral para el manejo de formularios, lo que obliga a los desarrolladores a crear sus propias implementaciones personalizadas o depender de bibliotecas menos capaces. Esto a menudo resulta en falta de consistencia, bajo rendimiento y mayor tiempo de desarrollo. TanStack Form busca abordar estos desafíos proporcionando una solución todo en uno para manejar formularios que es tanto poderosa como fácil de usar.

Con TanStack Form, los desarrolladores pueden abordar desafíos comunes relacionados con formularios, tales como:

- Enlace de datos reactivos y gestión de estado (state management)
- Validación compleja y manejo de errores
- Accesibilidad y diseño responsivo (responsive design)
- Internacionalización y localización
- Compatibilidad multiplataforma y estilos personalizados (custom styling)

Al proporcionar una solución completa para estos desafíos, TanStack Form permite a los desarrolladores construir formularios robustos y fáciles de usar con facilidad.

Lo mínimo indispensable para comenzar con TanStack Form es crear un formulario y agregar un campo. Tenga en cuenta que este ejemplo no incluye validación ni manejo de errores... por ahora.

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      fullName: '',
    },
    onSubmit: async ({ value }) => {
      // Do something with form data
      console.log(value)
    },
  }))
</script>

<div>
  <h1>Simple Form Example</h1>
  <form
    onsubmit={(e) => {
      e.preventDefault()
      e.stopPropagation()
      form.handleSubmit()
    }}
  >
    <div>
      <form.Field name="fullName">
        {#snippet children(field)}
          <input
            name={field.name}
            value={field.state.value}
            onblur={field.handleBlur}
            oninput={(e) => field.handleChange(e.target.value)}
          />
        {/snippet}
      </form.Field>
    </div>
    <button type="submit">Submit</button>
  </form>
</div>
```

A partir de aquí, estará listo para explorar todas las demás características de TanStack Form.
