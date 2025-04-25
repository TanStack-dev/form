---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:12:19.741Z'
id: form-validation
title: 表单验证
---
表单与字段验证 (Form and Field Validation)

TanStack Form 的核心功能之一是验证机制。TanStack Form 提供了高度可定制的验证方式：

- 您可以控制验证时机（值变更时、输入时、失焦时、提交时等）
- 验证规则可以在字段级别或表单级别定义
- 验证可以是同步或异步的（例如通过 API 调用）

## 验证触发时机

完全由您决定！`[tanstackField]` 指令接受一些回调属性如 `onChange` 或 `onBlur`。这些回调会接收字段当前值和 fieldAPI 对象，以便执行验证。如果发现验证错误，只需返回错误信息字符串，该信息将通过 `field.api.state.meta.errors` 获取。

示例如下：

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

上例中，验证在每次按键时触发（`onChange`）。如果改为失焦时验证，代码调整如下：

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
      <!-- 始终需要实现 onChange 以便 TanStack Form 接收变更 -->
      <!-- 监听字段的 onBlur 事件 -->
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

您可以通过实现不同的回调来控制验证时机，甚至可以在不同时间执行不同验证：

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
  }) => (value < 0 ? '无效值' : undefined)

  // ...
}
```

上例中，我们对同一字段在不同时机（按键时和失焦时）进行不同验证。由于 `field.state.meta.errors` 是数组，会显示当前所有相关错误。也可以通过 `field.state.meta.errorMap` 根据验证时机（onChange、onBlur 等）获取错误。下文将详细介绍错误显示。

## 错误显示

设置验证后，可以将错误从数组映射到 UI 中显示：

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

或使用 `errorMap` 属性访问特定错误：

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

需注意，`errors` 数组和 `errorMap` 的类型与验证器返回类型匹配：

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

如上所示，每个 `[tanstackField]` 通过 `onChange`、`onBlur` 等回调接受自己的验证规则。也可以通过向 `injectForm()` 函数传递类似回调来定义表单级验证规则（而非逐个字段定义）。

示例：

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
            <em
              >表单错误：{{ formErrorMap().onChange }}</em
            >
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
      // 以与字段相同的方式向表单添加验证器
      onChange({ value }) {
        if (value.age < 13) {
          return '必须年满13岁才能注册'
        }
        return undefined
      },
    },
  })

  // 订阅表单的 errorMap 以便更新时重新渲染
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

## 异步函数验证

虽然大多数验证可能是同步的，但许多情况下需要通过网络请求或其他异步操作进行验证。

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

同步和异步验证可以共存。例如，可以在同一字段上同时定义 `onBlur` 和 `onBlurAsync`：

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
    value < 13 ? '年龄必须至少13岁' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? '只能增加年龄' : undefined
  }

  // ...
}
```

同步验证方法（`onBlur`）会先执行，只有同步验证成功后才会执行异步方法（`onBlurAsync`）。要改变此行为，可将 `asyncAlways` 选项设为 `true`，这样无论同步方法结果如何都会执行异步方法。

### 内置防抖

虽然异步调用是验证数据库的有效方式，但每次按键都发起网络请求可能导致数据库遭受 DDoS 攻击。

我们通过添加单个属性提供简单的防抖方法：

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

这将为每个异步调用设置500毫秒的防抖延迟。还可以按验证属性单独覆盖此设置：

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

> 这将使 `onChangeAsync` 每1500毫秒执行一次，而 `onBlurAsync` 仍保持500毫秒间隔。

## 通过模式库验证

虽然函数提供了更灵活的验证方式，但可能较为冗长。为此，有些库提供基于模式的验证，可以大幅简化类型严格的验证。您还可以为整个表单定义单一模式并传递给表单级别，错误将自动传播到字段。

### 标准模式库

TanStack Form 原生支持所有遵循 [Standard Schema 规范](https://github.com/standard-schema/standard-schema) 的库，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意_：请确保使用模式库的最新版本，旧版本可能尚未支持 Standard Schema。

您可以像使用自定义函数一样将这些库的模式传递给 `validators` 属性：

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

表单和字段级别的异步验证同样支持：

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

## 阻止无效表单提交

表单提交时也会运行 `onChange`、`onBlur` 等回调，如果表单无效则会阻止提交。

表单状态对象有一个 `canSubmit` 标志，当任何字段无效且表单已被触碰时该标志为 false（即使某些字段根据其 `onChange`/`onBlur` 属性"技术上"无效，只要表单未被触碰，`canSubmit` 仍为 true）。

您可以通过 `injectStore` 订阅该值，例如用于在表单无效时禁用提交按钮（实践中禁用按钮不可访问，应使用 `aria-disabled`）。

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
