---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:26:05.640Z'
id: arrays
title: 配列
---

# 配列 (Arrays)

TanStack Formは、配列をフォームの値としてサポートしています。これには配列内のサブオブジェクト値も含まれます。

## 基本的な使い方

配列を使用するには、配列値に対して`field.state.value`を使用し、[`solid-js`の`Index`](https://www.solidjs.com/tutorial/flow_index)と組み合わせます:

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    people: [] as Array<{ age: number; name: string }>,
  },
  onSubmit: ({ value }) => alert(JSON.stringify(value)),
})
</script>

<template>
  <form.Field name="people">
    <template v-slot="{ field, state }">
      <div>
        <form.Field
          v-for="(_, i) of field.state.value"
          :key="i"
          :name="`people[${i}].name`"
        >
          <template v-slot="{ field: subField, state }">
            <!-- ... -->
          </template>
        </form.Field>
      </div>
    </template>
  </form.Field>
</template>
```

`field`に対して`pushValue`を実行するたびに、マップされたスロットが生成されます:

```vue
<button @click="field.pushValue({ name: '', age: 0 })" type="button">
  Add person
</button>
```

サブフィールドは以下のように使用できます:

```vue
<form.Field
  v-for="(_, i) of field.state.value"
  :key="i"
  :name="`people[${i}].name`"
>
  <template v-slot="{ field: subField, state }">
    <div>
      <label>
        <div>Name for person {{ i }}</div>
        <input
          :value="subField.state.value"
          @input="
          (e) =>
          subField.handleChange(
            (e.target as HTMLInputElement).value,
          )
          "
        />
      </label>
    </div>
  </template>
</form.Field>
```

## 完全な例

```vue
<script setup lang="ts">
import { useForm } from '@tanstack/vue-form'

const form = useForm({
  defaultValues: {
    people: [] as Array<{ age: number; name: string }>,
  },
  onSubmit: ({ value }) => alert(JSON.stringify(value)),
})
</script>

<template>
  <form
    @submit="
      (e) => {
        e.preventDefault()
        e.stopPropagation()
        form.handleSubmit()
      }
    "
  >
    <div>
      <form.Field name="people">
        <template v-slot="{ field, state }">
          <div>
            <form.Field
              v-for="(_, i) of field.state.value"
              :key="i"
              :name="`people[${i}].name`"
            >
              <template v-slot="{ field: subField, state }">
                <div>
                  <label>
                    <div>Name for person {{ i }}</div>
                    <input
                      :value="subField.state.value"
                      @input="
                        (e) =>
                          subField.handleChange(
                            (e.target as HTMLInputElement).value,
                          )
                      "
                    />
                  </label>
                </div>
              </template>
            </form.Field>

            <button
              @click="field.pushValue({ name: '', age: 0 })"
              type="button"
            >
              Add person
            </button>
          </div>
        </template>
      </form.Field>
    </div>
    <form.Subscribe>
      <template v-slot="{ canSubmit, isSubmitting }">
        <button type="submit" :disabled="!canSubmit">
          {{ isSubmitting ? '...' : 'Submit' }}
        </button>
      </template>
    </form.Subscribe>
  </form>
</template>
```
