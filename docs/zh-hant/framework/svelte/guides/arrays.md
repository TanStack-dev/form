---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-25T20:49:22.829Z'
id: arrays
title: 陣列
---

TanStack Form 支援將陣列 (array) 作為表單值使用，包含陣列中的子物件值。

## 基本用法

要使用陣列，你可以對陣列值使用 `field.state.value` 並搭配 [`each blocks`](https://svelte.dev/docs/svelte/each)：

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

這會在每次對 `field` 執行 `pushValue` 時重新生成列表：

```svelte
<button onclick={() => field.pushValue({ name: '', age: 0 })} type="button">
  新增人員
</button>
```

最後，你可以像這樣使用子欄位 (subfield)：

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

## 完整範例

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
                  <div>人員 {i} 的姓名</div>
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
          新增人員
        </button>
      </div>
    {/snippet}
  </form.Field>

  <button type="submit">提交</button>
</form>
```
