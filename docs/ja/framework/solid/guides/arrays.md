---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:27:21.116Z'
id: arrays
title: 配列
---

# 配列

TanStack Formは、配列をフォームの値としてサポートしており、配列内のサブオブジェクト値も扱うことができます。

## 基本的な使い方

配列を使用するには、配列値に対して`field.state.value`を使用し、[`solid-js`の`Index`](https://www.solidjs.com/tutorial/flow_index)と組み合わせます:

```jsx
function App() {
  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
  }))

  return (
    <form.Field name="people" mode="array">
      {(field) => (
        <Show when={field().state.value.length > 0}>
          {/* Do not change this to `For` or things will not work as-expected */}
          <Index each={field().state.value}>
            {
              (_, i) => null // ...
            }
          </Index>
        </Show>
      )}
    </form.Field>
  )
}
```

> `solid-js`の`Index`を使用する必要があり、`For`は使用できません。`For`を使用すると、配列が変更されるたびに内部コンポーネントが再レンダリングされてしまいます。
>
> これにより、フィールドが値を失い、サブフィールドの値が削除されてしまいます。

`field`に対して`pushValue`を実行するたびに、マップされたJSXが生成されます:

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  Add person
</button>
```

最後に、サブフィールドは次のように使用できます:

```jsx
<form.Field name={`people[${i}].name`}>
  {(subField) => (
    <input
      value={subField().state.value}
      onInput={(e) => {
        subField().handleChange(e.currentTarget.value)
      }}
    />
  )}
</form.Field>
```

## 完全な例

```jsx
function App() {
  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
    onSubmit: ({ value }) => alert(JSON.stringify(value)),
  }))

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault()
          e.stopPropagation()
          form.handleSubmit()
        }}
      >
        <form.Field name="people">
          {(field) => (
            <div>
              <Show when={field().state.value.length > 0}>
                {/* Do not change this to For or the test will fail */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>Name for person {i}</div>
                            <input
                              value={subField().state.value}
                              onInput={(e) => {
                                subField().handleChange(e.currentTarget.value)
                              }}
                            />
                          </label>
                        </div>
                      )}
                    </form.Field>
                  )}
                </Index>
              </Show>

              <button
                onClick={() => field().pushValue({ name: '', age: 0 })}
                type="button"
              >
                Add person
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">Submit</button>
      </form>
    </div>
  )
}
```
