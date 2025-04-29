---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-29T23:14:57.393Z'
id: arrays
title: 数组
---

TanStack Form 支持将数组作为表单值使用，包括数组内的子对象值。

## 基础用法

要使用数组，可以在数组值上使用 `field.state.value` 并结合 [`each blocks`](https://svelte.dev/docs/svelte/each)：

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

每次在 `field` 上调用 `pushValue` 时，列表会重新生成：

```svelte
<button onclick={() => field.pushValue({ name: '', age: 0 })} type="button">
  添加人员
</button>
```

最后，可以像这样使用子字段：

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

## 完整示例

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
                  <div>人员 {i} 的姓名</div>
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
          添加人员
        </button>
      </div>
    {/snippet}
  </form.Field>

  <button type="submit"> 提交 </button>
</form>
```
