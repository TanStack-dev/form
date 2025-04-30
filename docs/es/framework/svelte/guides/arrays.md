---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:50:25.714Z'
id: arrays
title: Arreglos
---

# Arrays

TanStack Form admite arrays como valores en un formulario, incluyendo valores de sub-objetos dentro de un array.

## Uso Básico

Para usar un array, puede utilizar `field.state.value` en un valor de array junto con [`each blocks`](https://svelte.dev/docs/svelte/each):

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
  }))
</script>

<form.Field name="people" mode="array">
  {#snippet children(field)}
    {#each field.state.value as person, i}
      <!-- ... -->
    {/each}
  {/snippet}
</form.Field>
```

Esto regenerará la lista cada vez que ejecute `pushValue` en `field`:

```svelte
<button onclick={() => field.pushValue({ name: '', age: 0 })} type="button">
  Añadir persona
</button>
```

Finalmente, puede usar un subcampo de la siguiente manera:

```svelte
<form.Field name={`people[${i}].name`}>
  {#snippet children(subField)}
    <input
      value={subField.state.value}
      oninput={(e) => {
        subField.handleChange(e.currentTarget.value)
      }}
    />
  {/snippet}
</form.Field>
```

## Ejemplo Completo

```svelte
<script lang="ts">
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      people: [] as Array<{ age: number; name: string }>,
    },
    onSubmit: ({ value }) => alert(JSON.stringify(value)),
  }))
</script>

<form
  id="form"
  onsubmit={(e) => {
    e.preventDefault()
    e.stopPropagation()
    form.handleSubmit()
  }}
>
  <form.Field name="people">
    {#snippet children(field)}
      <div>
        {#each field.state.value as person, i}
          <form.Field name={`people[${i}].name`}>
            {#snippet children(subField)}
              <div>
                <label>
                  <div>Nombre para la persona {i}</div>
                  <input
                    value={person.name}
                    oninput={(e: Event) => {
                      const target = e.target as HTMLInputElement
                      subField.handleChange(target.value)
                    }}
                  />
                </label>
              </div>
            {/snippet}
          </form.Field>
        {/each}

        <button
          onclick={() => field.pushValue({ name: '', age: 0 })}
          type="button"
        >
          Añadir persona
        </button>
      </div>
    {/snippet}
  </form.Field>

  <button type="submit"> Enviar </button>
</form>
```
