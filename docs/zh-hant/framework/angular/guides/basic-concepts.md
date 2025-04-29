---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-29T23:12:45.739Z'
id: basic-concepts
title: 基本概念
---

本頁介紹 `@tanstack/angular-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更有效地理解並運用此函式庫。

## 表單實例 (Form Instance)

表單實例是一個代表單一表單的物件，提供操作表單的方法與屬性。您可以使用 `injectForm` 函式建立表單實例。此函式接受包含 `onSubmit` 函式的物件，該函式會在表單提交時被呼叫。

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // 處理表單資料
    console.log(value)
  },
})
```

## 欄位 (Field)

欄位代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位透過 `tanstackField` 指令建立。此指令接受 `name` 屬性，該屬性應對應表單預設值中的鍵名。同時會公開一個名為 `field` 的指令內部實例，應透過模板變數存取欄位內部狀態。

範例：

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## 欄位狀態 (Field State)

每個欄位都有其專屬狀態，包含當前值、驗證狀態、錯誤訊息等中繼資料。可透過 `fieldApi.state` 屬性存取欄位狀態。

範例：

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三種欄位狀態特別有助於觀察使用者互動：當使用者點擊/切換至欄位時為「已觸碰 _(touched)_」、使用者未修改值時為「原始狀態 _(pristine)_」、值被修改後則為「已變更 _(dirty)_」。可透過以下標誌檢查這些狀態：

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## 欄位 API (Field API)

欄位 API 是透過建立欄位時 `tanstackField.api` 屬性存取的物件，提供操作欄位狀態的方法。

範例：

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value"
/>
```

## 驗證 (Validation)

`@tanstack/angular-form` 內建同步與非同步驗證功能。驗證函式可透過 `tanstackField` 指令的 `validators` 屬性傳遞。

範例：

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
      ? '必須填寫名字'
      : value.length < 3
        ? '名字至少需 3 個字元'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名字中不可包含 "error"'
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

## 標準結構描述驗證 (Validation with Standard Schema Libraries)

除了手動驗證選項外，我們也支援 [Standard Schema](https://github.com/standard-schema/standard-schema) 規範。

您可使用任何實作此規範的函式庫定義結構描述，並將其傳遞給表單或欄位驗證器。

支援的函式庫包括：

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

範例：

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
        onChange: z.string().min(3, '名字至少需 3 個字元'),
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
      message: "名字中不可包含 'error'",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // 處理表單資料
      console.log(value)
    },
  })

  z = z
}
```

## 響應式 (Reactivity)

`@tanstack/angular-form` 提供透過 `injectStore(this.form, selector)` 訂閱表單與欄位狀態變更的方式。

範例：

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## 監聽器 (Listeners)

`@tanstack/angular-form` 允許您針對特定觸發條件做出反應，並「監聽」這些條件以執行副作用。

範例：

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
    console.log(`國家已變更為: ${value}, 重置省份`)
    this.form.setFieldValue('province', '')
  }
```

更多資訊請參閱 [監聽器](./listeners.md)

## 陣列欄位 (Array Fields)

陣列欄位讓您能管理表單中的值列表，例如興趣清單。您可使用 `tanstackField` 指令建立陣列欄位。

操作陣列欄位時，可使用 `pushValue`、`removeValue` 和 `swapValues` 方法來新增、移除和交換陣列中的值。

範例：

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        興趣
        <div>
          @if (!hobbies.api.state.value.length) {
            尚未新增興趣
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">名稱:</label>
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
          新增興趣
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

以上是 `@tanstack/angular-form` 函式庫中使用的基本概念與術語。理解這些概念將幫助您更有效率地運用此函式庫，輕鬆建立複雜的表單。
