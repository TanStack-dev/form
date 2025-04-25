---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:11:24.303Z'
id: basic-concepts
title: 基本概念
---
本页介绍 `@tanstack/lit-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解并运用该库及其与 Lit 的配合使用。

## 表单选项 (Form Options)

您可以通过 `formOptions` 函数为表单创建可复用的配置选项，以便在多个表单间共享。

例如：

```tsx
const formOpts = formOptions({
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

## 表单实例 (Form Instance)

表单实例是表示单个表单的对象，提供操作表单的方法和属性。您可以通过 `@tanstack/lit-form` 提供的 `TanStackFormController` 接口创建表单实例。`TanStackFormController` 通过当前表单所属类（`this`）和默认表单选项进行实例化，负责初始化表单状态、处理表单提交，并提供管理表单字段及其验证的方法。

```tsx
#form = new TanStackFormController(this, {
  defaultValues: {
    firstName: '',
    lastName: '',
    employed: false,
    jobTitle: '',
  } as Employee,
})
```

您也可以不使用 `formOptions`，直接通过独立的 `TanStackFormController` API 创建表单实例：

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## 字段 (Field)

字段表示单个表单输入元素（如文本输入框或复选框）。通过表单实例提供的 `field(FieldOptions, callback)` 方法创建字段。该方法接收一个 `FieldOptions` 对象和一个回调函数，回调函数会获得一个 `FieldApi` 对象。该对象提供获取字段当前值、处理输入变更和处理失焦事件的方法。

例如：

```ts
 ${this.#form.field(
    {
      name: `firstName`,
      validators: {
        onChange: ({ value }) =>
          value.length < 3 ? "Not long enough" : undefined,
        },
      },
      (field: FieldApi<Employee, "firstName">) => {
        return html` <div>
          <label class="first-name-label">First Name</label>
          <input
           id="firstName"
           type="text"
           class="first-name-input"
           placeholder="First Name"
           @blur="${() => field.handleBlur()}"
           .value="${field.state.value}"
           @input="${(event: InputEvent) => {
           if (event.currentTarget) {
            const newValue = (event.currentTarget as HTMLInputElement).value;
            field.handleChange(newValue);
           }
          }}"
        />
      </div>`;
    },
)}
```

## 字段状态 (Field State)

每个字段都有独立的状态，包含当前值、验证状态、错误信息等元数据。可通过 `field.state` 属性访问字段状态。

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三种字段状态能有效反映用户交互行为：当用户点击/聚焦字段时为 _"已触碰 (touched)"_，用户未修改字段值时保持 _"原始状态 (pristine)"_，值变更后转为 _"已修改 (dirty)"_。可通过如下所示的 `isTouched`、`isPristine` 和 `isDirty` 标志位检测这些状态：

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![字段状态示意图](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
