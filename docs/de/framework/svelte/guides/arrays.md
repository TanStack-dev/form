---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:56:11.954Z'
id: arrays
title: Arrays
---

TanStack Form unterstützt Arrays als Werte in einem Formular, einschließlich Unterobjekt-Werten innerhalb eines Arrays.

## Grundlegende Verwendung

Um ein Array zu verwenden, können Sie `field.state.value` auf einen Array-Wert in Kombination mit [`each blocks`](https://svelte.dev/docs/svelte/each) anwenden:

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

Dadurch wird die Liste jedes Mal neu generiert, wenn Sie `pushValue` auf `field` ausführen:

```svelte
<button onclick={() => field.pushValue({ name: '', age: 0 })} type="button">
  Person hinzufügen
</button>
```

Schließlich können Sie ein Unterfeld wie folgt verwenden:

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

## Vollständiges Beispiel

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
                  <div>Name für Person {i}</div>
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
          Person hinzufügen
        </button>
      </div>
    {/snippet}
  </form.Field>

  <button type="submit"> Absenden </button>
</form>
```
