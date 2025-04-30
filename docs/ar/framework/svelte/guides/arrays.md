---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T22:07:45.709Z'
id: arrays
title: المصفوفات
---

يدعم TanStack Form المصفوفات كقيم في النموذج، بما في ذلك القيم الفرعية داخل المصفوفة.

## الاستخدام الأساسي

لاستخدام مصفوفة، يمكنك استخدام `field.state.value` على قيمة مصفوفة بالاقتران مع [`each blocks`](https://svelte.dev/docs/svelte/each):

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

سيؤدي هذا إلى إعادة إنشاء القائمة في كل مرة تقوم فيها باستدعاء `pushValue` على `field`:

```svelte
<button onclick={() => field.pushValue({ name: '', age: 0 })} type="button">
  Add person
</button>
```

أخيرًا، يمكنك استخدام حقل فرعي كما يلي:

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

## مثال كامل

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
                  <div>Name for person {i}</div>
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
          Add person
        </button>
      </div>
    {/snippet}
  </form.Field>

  <button type="submit"> Submit </button>
</form>
```
