---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:11:17.959Z'
id: basic-concepts
title: 基本概念
---
本页面介绍了 `@tanstack/angular-form` 库中使用的基本概念和术语。熟悉这些概念将帮助您更好地理解和使用该库。

## 表单实例 (Form Instance)

表单实例 (Form Instance) 是一个表示单个表单的对象，提供了操作表单的方法和属性。您可以使用 `injectForm` 函数创建表单实例。该函数接收一个包含 `onSubmit` 函数的对象，该函数在表单提交时被调用。

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // 处理表单数据
    console.log(value)
  },
})
```

## 字段 (Field)

字段 (Field) 表示单个表单输入元素，例如文本输入框或复选框。字段通过 `tanstackField` 指令创建。该指令接收一个 name 属性，该属性应与表单默认值中的键匹配。它还暴露了一个名为 `field` 的指令内部实例，应通过模板变量访问字段内部状态。

示例：

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## 字段状态 (Field State)

每个字段都有自己的状态 (Field State)，包括当前值、验证状态、错误消息和其他元数据。您可以通过 `fieldApi.state` 属性访问字段状态。

示例：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三种字段状态可以非常直观地反映用户与字段的交互情况：当用户点击/聚焦到字段时，字段状态为 _"touched"_；在用户修改值之前，字段状态为 _"pristine"_；值被修改后，字段状态变为 _"dirty"_。您可以通过 `isTouched`、`isPristine` 和 `isDirty` 标志来检查这些状态，如下所示。

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![字段状态](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 字段 API (Field API)

字段 API (Field API) 是一个对象，在创建字段时通过 `tanstackField.api` 属性访问。它提供了操作字段状态的方法。

示例：

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## 验证 (Validation)

`@tanstack/angular-form` 开箱即用地支持同步和异步验证。验证函数可以通过 `validators` 属性传递给 `tanstackField` 指令。

示例：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="firstName" #firstName="field">
      <input
        [value]="firstName.api.state.value"
        (blur)="firstName.api.handleBlur()"
        (input)="firstName.api.handleChange($any($event).target.value)"
      />
    </ng-container>
  `,
})
export class AppComponent {
  firstNameValidator: FieldValidateFn<any, any, string, any> = ({
                                                                       value,
                                                                     }) =>
    !value
      ? '必须填写名字'
      : value.length < 3
        ? '名字至少需要 3 个字符'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名字中不能包含 "error"'
    }

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      console.log(value)
    },
  })
}
```

## 使用标准模式库验证 (Validation with Standard Schema Libraries)

除了手动编写的验证选项外，我们还支持 [标准模式 (Standard Schema)](https://github.com/standard-schema/standard-schema) 规范。

您可以使用实现该规范的任何库定义模式，并将其传递给表单或字段验证器。

支持的库包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

示例：

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="firstName"
      [validators]="{
        onChange: z.string().min(3, '名字至少需要 3 个字符'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: firstNameAsyncValidator
      }"
      #firstName="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  firstNameAsyncValidator = z.string().refine(
    async (value) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return !value.includes('error')
    },
    {
      message: "名字中不能包含 'error'",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // 处理表单数据
      console.log(value)
    },
  })

  z = z
}
```

## 响应式 (Reactivity)

`@tanstack/angular-form` 提供了一种通过 `injectStore(this.form, selector)` 订阅表单和字段状态变化的方式。

示例：

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## 监听器 (Listeners)

`@tanstack/angular-form` 允许您响应特定触发器并"监听"它们以执行副作用。

示例：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="country"
      [listeners]="{
        onChange: onCountryChange
      }"
      #country="field"
    ></ng-container>
  `,
})

...

onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`国家已更改为: ${value}, 重置省份`)
    this.form.setFieldValue('province', '')
  }
```

更多信息请参阅 [监听器](./listeners.md)

## 数组字段 (Array Fields)

数组字段 (Array Fields) 允许您管理表单中的值列表，例如爱好列表。您可以使用 `tanstackField` 指令创建数组字段。

在处理数组字段时，可以使用字段的 `pushValue`、`removeValue` 和 `swapValues` 方法来添加、删除和交换数组中的值。

示例：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        爱好
        <div>
          @if (!hobbies.api.state.value.length) {
            未找到爱好
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">名称:</label>
                  <input
                    [id]="hobbyName.api.name"
                    [name]="hobbyName.api.name"
                    [value]="hobbyName.api.state.value"
                    (blur)="hobbyName.api.handleBlur()"
                    (input)="
                      hobbyName.api.handleChange($any($event).target.value)
                    "
                  />
                  <button
                    type="button"
                    (click)="hobbies.api.removeValue($index)"
                  >
                    X
                  </button>
                </div>
              </ng-container>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyDesc($index)"
                #hobbyDesc="field"
              >
                <div>
                  <label [for]="hobbyDesc.api.name">描述:</label>
                  <input
                    [id]="hobbyDesc.api.name"
                    [name]="hobbyDesc.api.name"
                    [value]="hobbyDesc.api.state.value"
                    (blur)="hobbyDesc.api.handleBlur()"
                    (input)="
                      hobbyDesc.api.handleChange($any($event).target.value)
                    "
                  />
                </div>
              </ng-container>
            </div>
          }
        </div>
        <button type="button" (click)="hobbies.api.pushValue(defaultHobby)">
          添加爱好
        </button>
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  defaultHobby = {
    name: '',
    description: '',
    yearsOfExperience: 0,
  }

  getHobbyName = (idx: number) => `hobbies[${idx}].name` as const;
  getHobbyDesc = (idx: number) => `hobbies[${idx}].description` as const;

  form = injectForm({
    defaultValues: {
      hobbies: [] as Array<{
        name: string
        description: string
        yearsOfExperience: number
      }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

这些是 `@tanstack/angular-form` 库中使用的基本概念和术语。理解这些概念将帮助您更高效地使用该库，轻松创建复杂的表单。
