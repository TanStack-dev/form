---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-29T23:30:23.418Z'
id: form-validation
title: 表單驗證
---

# 表單與欄位驗證 (Form and Field Validation)

TanStack Form 的核心功能之一就是驗證機制。TanStack Form 讓驗證高度可自訂：

- 你可以控制驗證的時機（在變更時、輸入時、失焦時、提交時...）
- 驗證規則可以在欄位層級或表單層級定義
- 驗證可以是同步或非同步的（例如 API 呼叫的結果）

## 驗證何時執行？

由你決定！`[tanstackField]` 指令接受一些回調函式作為 props，例如 `onChange` 或 `onBlur`。這些回調會接收欄位的當前值以及 fieldAPI 物件，讓你可以執行驗證。如果發現驗證錯誤，只需返回錯誤訊息字串，它就會出現在 `field.api.state.meta.errors` 中。

以下是範例：

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

在上面的範例中，驗證在每次鍵入時執行（`onChange`）。如果我們希望驗證在欄位失焦時執行，可以這樣修改程式碼：

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
      <!-- 我們始終需要實作 onChange，讓 TanStack Form 接收變更 -->
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

因此，你可以透過實作所需的回調來控制驗證的時機。你甚至可以在不同時間執行不同的驗證：

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
      <!-- 我們始終需要實作 onChange，讓 TanStack Form 接收變更 -->
      <!-- 監聽欄位的 onBlur 事件 -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (!age.api.state.meta.isValid) {
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

在上面的範例中，我們在同一欄位的不同時間（每次鍵入和欄位失焦時）驗證不同內容。由於 `field.state.meta.errors` 是一個陣列，所有相關錯誤都會在特定時間顯示。你也可以使用 `field.state.meta.errorMap` 根據驗證時機（onChange、onBlur 等）取得錯誤。更多關於顯示錯誤的資訊如下。

## 顯示錯誤

設定好驗證後，你可以將錯誤從陣列映射到 UI 中顯示：

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

或者使用 `errorMap` 屬性來存取特定錯誤：

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

值得一提的是，我們的 `errors` 陣列和 `errorMap` 與驗證器返回的類型相匹配。這意味著：

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

如上所示，每個 `[tanstackField]` 透過 `onChange`、`onBlur` 等回調接受自己的驗證規則。也可以透過將類似的回調傳遞給 `injectForm()` 函式，在表單層級（而非逐個欄位）定義驗證規則。

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
      // 像添加欄位驗證一樣添加表單驗證
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

### 從表單驗證器設置欄位層級錯誤

你可以從表單驗證器中設置欄位錯誤。一個常見的使用情境是在表單的 `onSubmitAsync` 驗證器中呼叫單一 API 端點來驗證所有欄位。

```angular-ts
@Component({
  selector: 'app-root',
  imports: [TanStackField],
  template: `
    <form (submit)="handleSubmit($event)">
      <div>
        <ng-container
          [tanstackField]="form"
          name="age"
          #ageField="field"
        >
          <label [for]="ageField.api.name">Age:</label>
          <input
            type="number"
            [name]="ageField.api.name"
            [value]="ageField.api.state.value"
            (blur)="ageField.api.handleBlur()"
            (input)="ageField.api.handleChange($any($event).target.valueAsNumber)"
          />
          @if (ageField.api.state.meta.errors.length > 0) {
            <em role="alert">{{ ageField.api.state.meta.errors.join(', ') }}</em>
          }
        </ng-container>
      </div>
      <button type="submit">Submit</button>
    </form>
  `,
})

export class AppComponent {
  form = injectForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // 在伺服器上驗證值
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // `form` 鍵是可選的
            fields: {
              age: 'Must be 13 or older to sign',
              // 使用欄位名稱設置嵌套欄位的錯誤
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
            },
          };
        }

        return null;
      },
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
    this.form.handleSubmit();
  }
}
```

> 值得一提的是，如果你有一個表單驗證函式返回錯誤，該錯誤可能會被欄位特定的驗證覆蓋。
>
> 這意味著：
>
> ```angular-ts
> @Component({
>   selector: 'app-root',
>   standalone: true,
>   imports: [TanStackField],
>   template: `
>       <div>
>         <ng-container
>          [tanstackField]="form"
>         name="age"
>        #ageField="field"
>       [validators]="{
>        onChange: fieldValidator
>     }"
>  >
>   <input type="number" [value]="ageField.api.state.value"
>   (input)="ageField.api.handleChange($any($event).target.valueAsNumber)"
>   />
>    @if (ageField.api.state.meta.errors.length > 0) {
>       <em role="alert">{{ ageField.api.state.meta.errors.join(', ') }}</em>
>     }
>   </ng-container>
> </div>
> `,
> })
> export class AppComponent {
>   form = injectForm({
>     defaultValues: {
>       age: 0,
>     },
>     validators: {
>       onChange: ({ value }) => {
>         return {
>           fields: {
>             age: value.age < 12 ? 'Too young!' : undefined,
>           },
>         };
>       },
>     },
>   });
>
>   fieldValidator: FieldValidateFn<any, any, number> = ({ value }) =>
>     value % 2 === 0 ? 'Must be odd!' : undefined;
> }
>
> ```
>
> 即使表單層級驗證返回了 'Too young!' 錯誤，也只會顯示 `'Must be odd!'`。

## 非同步函式驗證

雖然我們預期大多數驗證會是同步的，但在許多情況下，網路呼叫或其他非同步操作會很有用。

為此，我們提供了專用的 `onChangeAsync`、`onBlurAsync` 等方法來進行驗證：

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

同步和非同步驗證可以共存。例如，可以在同一欄位上定義 `onBlur` 和 `onBlurAsync`：

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

同步驗證方法（`onBlur`）會先執行，只有當同步方法（`onBlur`）成功時才會執行非同步方法（`onBlurAsync`）。要改變此行為，請將 `asyncAlways` 選項設為 `true`，這樣無論同步方法的結果如何，都會執行非同步方法。

### 內建防抖動 (Debouncing)

雖然非同步呼叫是驗證資料庫的正確方式，但在每次鍵入時執行網路請求是對資料庫進行 DDoS 攻擊的好方法。

相反地，我們提供了一個簡單的方法來防抖動你的 `async` 呼叫，只需添加一個屬性：

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

這將以 500 毫秒的延遲防抖動每個非同步呼叫。你甚至可以針對每個驗證屬性覆寫此設定：

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

## 透過結構描述函式庫驗證

雖然
