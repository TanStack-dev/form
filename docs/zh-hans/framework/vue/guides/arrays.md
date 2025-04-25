---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:10:04.251Z'
id: arrays
title: 数组
---
TanStack Form 支持将数组作为表单值使用，包括数组内的子对象值。

## 基本用法

要使用数组，你可以对数组值使用 `field.state.value`，并结合 [`solid-js` 的 `Index`](https://www.solidjs.com/tutorial/flow_index)：

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

每次在 `field` 上调用 `pushValue` 时都会生成映射的插槽：

```vue
<button @click="field.pushValue({ name: '', age: 0 })" type="button">
  添加人员
</button>
```

最后，可以像这样使用子字段：

```vue
<form.Field
  v-for="(_, i) of field.state.value"
  :key="i"
  :name="`people[${i}].name`"
>
  <template v-slot="{ field: subField, state }">
    <div>
      <label>
        <div>人员 {{ i }} 的姓名</div>
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

## 完整示例

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
                    <div>人员 {{ i }} 的姓名</div>
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
              添加人员
            </button>
          </div>
        </template>
      </form.Field>
    </div>
    <form.Subscribe>
      <template v-slot="{ canSubmit, isSubmitting }">
        <button type="submit" :disabled="!canSubmit">
          {{ isSubmitting ? '...' : '提交' }}
        </button>
      </template>
    </form.Subscribe>
  </form>
</template>
```
