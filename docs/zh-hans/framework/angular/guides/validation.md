---
source-updated-at: '2025-04-16T14:15:06.000Z'
translation-updated-at: '2025-04-29T23:16:26.193Z'
id: form-validation
title: 表单验证
---

TanStack Form 的核心功能之一是验证 (validation) 概念。它提供了高度可定制的验证机制：

- 可控制验证触发时机（值变更时、输入时、失焦时、提交时等）
- 验证规则可在字段级或表单级定义
- 支持同步或异步验证（如通过 API 调用）

## 验证触发时机

完全由您掌控！`[tanstackField]` 指令接收如 `onChange` 或 `onBlur` 等回调函数作为 props。这些回调会接收字段当前值和 fieldAPI 对象，便于执行验证。若发现错误，只需返回字符串类型的错误信息，该信息将通过 `field.api.state.meta.errors` 获取。

示例：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">年龄：</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '必须年满13岁才能注册' : undefined

  // ...
}
```

上例在每次输入时触发验证（`onChange`）。若需改为失焦时验证，代码调整如下：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onBlur: ageValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">年龄：</label>
      <!-- 仍需保留 onChange 以保证 TanStack Form 能接收变更 -->
      <!-- 监听字段的失焦事件 -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)='age.api.handleBlur()'
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '必须年满13岁才能注册' : undefined

  // ...
}
```

通过实现不同的回调函数，您可以灵活控制验证时机。甚至可以在不同时机执行不同验证：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator,
        onBlur: minimumAgeValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">年龄：</label>
      <!-- 仍需保留 onChange 以保证 TanStack Form 能接收变更 -->
      <!-- 监听字段的失焦事件 -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '必须年满13岁才能注册' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? '无效数值' : undefined)

  // ...
}
```

上例展示了同一字段在不同时机（输入时和失焦时）的差异化验证。由于 `field.state.meta.errors` 是数组，会显示当前所有相关错误。也可通过 `field.state.meta.errorMap` 按验证时机（onChange/onBlur 等）获取错误。下文将详述错误展示。

## 错误展示

配置验证后，可将错误数组映射到界面展示：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '必须年满13岁才能注册' : undefined

  // ...
}
```

或使用 `errorMap` 属性获取特定错误：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      @if (age.api.state.meta.errorMap['onChange']) {
        <em role="alert">{{ age.api.state.meta.errorMap['onChange'] }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '必须年满13岁才能注册' : undefined

  // ...
}
```

需注意 `errors` 数组和 `errorMap` 的类型与验证器返回类型一致：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      <!-- errorMap.onChange 类型为 `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors 类型为 `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">用户年龄不足</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '必须年满13岁才能注册' : undefined

  // ...
}
```

## 字段级与表单级验证

如上所示，每个 `[tanstackField]` 可通过 `onChange`、`onBlur` 等回调定义独立验证规则。也可通过 `injectForm()` 函数在表单级定义验证规则：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <div>
      <ng-container [tanstackField]="form" name="age" #age="field">
        <!-- ... -->
        @if (formErrorMap().onChange) {
          <div>
            <em>表单错误：{{ formErrorMap().onChange }}</em>
          </div>
        }
        <!-- ... -->
      </ng-container>
    </div>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      age: 0,
    },
    onSubmit({ value }) {
      console.log(value)
    },
    validators: {
      // 表单级验证器添加方式与字段级相同
      onChange({ value }) {
        if (value.age < 13) {
          return '必须年满13岁才能注册'
        }
        return undefined
      },
    },
  })

  // 订阅表单的 errorMap 以保证更新时重新渲染
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

## 异步函数验证

虽然大多数验证是同步的，但网络请求等异步操作也常被用于验证。

为此，我们提供了专用的 `onChangeAsync`、`onBlurAsync` 等方法：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onChangeAsync: ageValidator }"
      #age="field"
    >
      <label [for]="age.api.name">年龄：</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return value < 13 ? '必须年满13岁才能注册' : undefined
  }

  // ...
}
```

同步与异步验证可共存。例如同一字段可同时定义 `onBlur` 和 `onBlurAsync`：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onBlur: ensureAge13, onBlurAsync: ensureOlderAge }"
      #age="field"
    >
      <label [for]="age.api.name">年龄：</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type='number'
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.value)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ensureAge13: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? '年龄不得小于13岁' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? '只能增加年龄' : undefined
  }

  // ...
}
```

同步验证方法（`onBlur`）会优先执行，仅当其通过后才会执行异步方法（`onBlurAsync`）。如需改变此行为，可将 `asyncAlways` 选项设为 `true`，此时无论同步方法结果如何都会执行异步方法。

### 内置防抖

虽然异步验证适合数据库校验，但每次输入都发起网络请求可能导致数据库过载。

我们通过简单属性实现异步调用的防抖：

```angular-html
<ng-container
  [tanstackField]="form"
  name="age"
  asyncDebounceMs={500}
  [validators]="{ onChangeAsync: someValidator }"
  #age="field"
>
  <!-- ... -->
</ng-container>
```

这将使所有异步调用延迟500ms执行。还可针对特定验证单独设置：

```angular-html
<ng-container
  [tanstackField]="form"
  name="age"
  [validators]="{
    onChangeAsyncDebounceMs: 1500,
    onChangeAsync: someValidator,
    onBlurAsync: otherValidator
  }"
  #age="field"
>
  <!-- ... -->
</ng-container>
```

> 此时 `onChangeAsync` 每1500ms执行，而 `onBlurAsync` 仍保持500ms间隔。

## 模式库验证

虽然函数验证更灵活，但可能较冗长。为此，模式库 (schema libraries) 提供了简洁的类型安全验证方案。您还可以为整个表单定义单一模式，错误会自动传播到各字段。

### 标准模式库

TanStack Form 原生支持所有符合 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，主要包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 请确保使用模式库的最新版本，旧版本可能不支持 Standard Schema。

使用这些库的模式时，可像普通函数一样传入 `validators` props：

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: z.number().gte(13, '必须年满13岁才能注册'),
      }"
      #age="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  form = injectForm({
    // ...
   })

  z = z

  // ...
}
```

表单级和字段级的异步验证同样支持：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: z.number().gte(13, '必须年满13岁才能注册'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: increaseAge
      }"
      #age="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  increaseAge = z.number().refine(
    async (value) => {
      const currentAge = await fetchCurrentAgeOnProfile()
      return value >= currentAge
    },
    {
      message: '只能增加年龄',
    },
  )

  // ...
}
```

如需更精细控制，可将 Standard Schema 与回调函数结合：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onChangeAsync: ageValidator }"
      #age="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateAsyncFn<any, string, number> = async ({
    value,
    fieldApi,
  }) => {
    const errors = fieldApi.parseValueWithSchema(
      z.number().gte(13, '必须年满13岁才能注册'),
    )
    if (errors) return errors

    // 继续其他验证逻辑
  }

  // ...
}
```

## 阻止无效表单提交

表单提交时也会触发 `onChange`、`onBlur` 等回调，若表单无效将阻止提交。

表单状态对象包含 `canSubmit` 标志位：当任何字段无效且表单已被触碰时（即使某些字段根据 `onChange`/`onBlur` 规则"技术上"无效），该标志为 false（表单未被触碰前始终为 true）。

可通过 `injectStore` 订阅该值，例如用于在表单无效时禁用提交按钮（实践中应使用 `aria-disabled` 而非禁用按钮以保证可访问性）：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <!-- ... -->
    <button type="submit" [disabled]="!canSubmit()">
      {{ isSubmitting() ? '提交中...' : '提交' }}
    </button>
  `,
})
export class AppComponent {
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)

  // ...
}
```
