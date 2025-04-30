---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:07:20.070Z'
id: arrays
title: المصفوفات
---

يدعم TanStack Form المصفوفات كقيم في النموذج، بما في ذلك قيم الكائنات الفرعية داخل المصفوفة.

## الاستخدام الأساسي

لاستخدام مصفوفة، يمكنك استخدام `field.state.value` على قيمة مصفوفة بالاقتران مع [`Index` من `solid-js`](https://www.solidjs.com/tutorial/flow_index):

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
          {/* لا تغير هذا إلى `For` وإلا لن تعمل الأمور كما هو متوقع */}
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

> يجب عليك استخدام `Index` من `solid-js` وليس `For` لأن `For` سيتسبب في إعادة عرض المكونات الداخلية في كل مرة تتغير فيها المصفوفة.
>
> هذا يؤدي إلى فقدان الحقل لقيمته وبالتالي حذف قيمة الحقل الفرعي.

سيؤدي هذا إلى إنشاء JSX المعين في كل مرة تقوم فيها باستدعاء `pushValue` على `field`:

```jsx
<button onClick={() => field().pushValue({ name: '', age: 0 })} type="button">
  إضافة شخص
</button>
```

أخيرًا، يمكنك استخدام حقل فرعي كما يلي:

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

## مثال كامل

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
                {/* لا تغير هذا إلى For وإلا سيفشل الاختبار */}
                <Index each={field().state.value}>
                  {(_, i) => (
                    <form.Field name={`people[${i}].name`}>
                      {(subField) => (
                        <div>
                          <label>
                            <div>الاسم للشخص {i}</div>
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
                إضافة شخص
              </button>
            </div>
          )}
        </form.Field>
        <button type="submit">إرسال</button>
      </form>
    </div>
  )
}
```
