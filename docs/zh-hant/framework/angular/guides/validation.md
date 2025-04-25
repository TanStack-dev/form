---
source-updated-at: '2025-04-16T14:15:06.000Z'
translation-updated-at: '2025-04-25T20:47:24.792Z'
id: form-validation
title: 表單驗證
---
TanStack Form 的核心功能之一就是驗證 (validation) 概念。TanStack Form 讓驗證變得高度可自訂：

- 您可以控制何時執行驗證（變更時、輸入時、失焦時、提交時...）
- 驗證規則可以在欄位 (field) 層級或表單 (form) 層級定義
- 驗證可以是同步或非同步的（例如 API 呼叫的結果）

## 驗證何時執行？

由您決定！`[tanstackField]` 指令接受一些回調函式作為 props，例如 `onChange` 或 `onBlur`。這些回調函式會接收欄位的當前值以及 fieldAPI 物件，讓您可以執行驗證。如果發現驗證錯誤，只需將錯誤訊息作為字串返回，它就會出現在 `field.api.state.meta.errors` 中。

以下是一個範例：

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
      <label [for]="age.api.name">Age:</label>
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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

在上面的範例中，驗證在每次按鍵時執行 (`onChange`)。如果我們希望驗證在欄位失焦時執行，可以這樣修改程式碼：

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
      <label [for]="age.api.name">Age:</label>
      <!-- 我們始終需要實作 onChange，以便 TanStack Form 接收變更 -->
      <!-- 監聽欄位的 onBlur 事件 -->
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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

因此，您可以透過實作所需的回調函式來控制驗證的執行時機。您甚至可以在不同時間執行不同的驗證：

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
      <label [for]="age.api.name">Age:</label>
      <!-- 我們始終需要實作 onChange，以便 TanStack Form 接收變更 -->
      <!-- 監聽欄位的 onBlur 事件 -->
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
    value < 13 ? 'You must be 13 to make an account' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? 'Invalid value' : undefined)

  // ...
}
```

在上面的範例中，我們對同一個欄位在不同時間（每次按鍵和欄位失焦時）驗證不同內容。由於 `field.state.meta.errors` 是一個陣列，所有相關錯誤都會在給定時間顯示。您也可以使用 `field.state.meta.errorMap` 根據驗證執行時機（onChange、onBlur 等）取得錯誤。更多關於顯示錯誤的資訊如下。

## 顯示錯誤

設定好驗證後，您可以將錯誤從陣列映射到 UI 中顯示：

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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

或使用 `errorMap` 屬性來存取特定錯誤：

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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

值得一提的是，我們的 `errors` 陣列和 `errorMap` 會匹配驗證器返回的類型。這意味著：

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
      <!-- errorMap.onChange 的類型是 `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors 的類型是 `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">The user is not old enough</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

## 欄位層級與表單層級的驗證

如上所示，每個 `[tanstackField]` 透過 `onChange`、`onBlur` 等回調函式接受自己的驗證規則。也可以透過將類似的回調函式傳遞給 `injectForm()` 函式，在表單層級（而不是逐個欄位）定義驗證規則。

範例：

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
              >There was an error on the form: {{ formErrorMap().onChange }}</em
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
      // 以與欄位相同的方式向表單添加驗證器
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // 訂閱表單的 errorMap，以便更新時重新渲染
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

## 非同步函式驗證

雖然我們認為大多數驗證會是同步的，但在許多情況下，透過網路呼叫或其他非同步操作進行驗證會很有用。

為此，我們提供了專用的 `onChangeAsync`、`onBlurAsync` 等方法：

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
      <label [for]="age.api.name">Last Name:</label>
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
    return value < 13 ? 'You must be 13 to make an account' : undefined
  }

  // ...
}
```

同步和非同步驗證可以共存。例如，可以在同一個欄位上定義 `onBlur` 和 `onBlurAsync`：

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
      <label [for]="age.api.name">Last Name:</label>
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
    value < 13 ? 'You must be at least 13' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? 'You can only increase the age' : undefined
  }

  // ...
}
```

同步驗證方法 (`onBlur`) 會先執行，非同步方法 (`onBlurAsync`) 只有在同步方法 (`onBlur`) 成功時才會執行。要改變此行為，請將 `asyncAlways` 選項設為 `true`，這樣無論同步方法的結果如何，非同步方法都會執行。

### 內建防抖 (Debouncing)

雖然非同步呼叫是針對資料庫進行驗證的方式，但在每次按鍵時發送網路請求很容易導致資料庫遭受 DDoS 攻擊。

相反地，我們提供了一個簡單的方法來防抖您的 `async` 呼叫，只需添加一個屬性：

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

這將對每個非同步呼叫進行 500 毫秒的防抖處理。您甚至可以針對每個驗證屬性覆寫此設定：

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

> 這將每 1500 毫秒執行一次 `onChangeAsync`，而 `onBlurAsync` 將每 500 毫秒執行一次。

## 透過結構描述 (Schema) 函式庫進行驗證

雖然函式提供了更多彈性和自訂驗證的方式，但它們可能有點冗長。為了解決這個問題，有一些函式庫提供了基於結構描述的驗證，使簡寫和類型嚴格的驗證變得更加容易。您還可以為整個表單定義一個結構描述並將其傳遞到表單層級，錯誤會自動傳播到欄位。

### 標準結構描述函式庫

TanStack Form 原生支援所有遵循 [Standard Schema 規範](https://github.com/standard-schema/standard-schema) 的函式庫，最著名的包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

_注意：_ 請確保使用結構描述函式庫的最新版本，因為舊版本可能尚未支援 Standard Schema。

要使用這些函式庫的結構描述，您可以像自訂函式一樣將它們傳遞給 `validators` props：

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
        onChange: z.number().gte(13, 'You must be 13 to make an account'),
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

表單和欄位層級的非同步驗證也同樣支援：

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
        onChange: z.number().gte(13, 'You must be 13 to make an account'),
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
      message: 'You can only increase the age',
    },
  )

  // ...
}
```

如果您需要對 Standard Schema 驗證進行更多控制，可以像這樣將 Standard Schema 與回調函式結合使用：

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
      z.number().gte(13, 'You must be 13 to make an account'),
    )
    if (errors) return errors

    // 繼續進行您的驗證
  }

  // ...
}
```

## 防止提交無效表單

`onChange`、`onBlur` 等回調函式也會在提交表單時執行，如果表單無效，提交會被阻止。

表單狀態物件有一個 `canSubmit` 標誌，當任何欄位無效且表單已被觸碰時，該標誌為 false（即使某些欄位根據其 `onChange`/`onBlur` props「技術上」無效，`canSubmit` 在表單被觸碰之前仍
