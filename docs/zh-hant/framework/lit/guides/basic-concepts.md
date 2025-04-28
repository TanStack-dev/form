---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-25T20:47:50.231Z'
id: basic-concepts
title: 基本概念
---

本頁介紹 `@tanstack/lit-form` 函式庫中使用的基本概念與術語。熟悉這些概念將幫助您更好地理解並運用該函式庫及其與 Lit 的整合方式。

## 表單選項 (Form Options)

您可以使用 `formOptions` 函式建立表單選項，以便在多個表單之間共享配置。

範例：

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

## 表單實例 (Form Instance)

表單實例是一個代表單一表單的物件，提供操作表單的方法與屬性。您可以使用 `@tanstack/lit-form` 提供的 `TanStackFormController` 介面來建立表單實例。`TanStackFormController` 會以當前表單所屬的類別 (`this`) 與預設表單選項進行實例化，負責初始化表單狀態、處理表單提交，並提供管理表單欄位及其驗證的方法。

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

您也可以不使用 `formOptions`，直接透過獨立的 `TanStackFormController` API 建立表單實例：

```tsx
#form = new TanStackFormController(this, {
  ...formOpts,
})
```

## 欄位 (Field)

欄位代表單一表單輸入元素，例如文字輸入框或核取方塊。欄位是透過表單實例提供的 `field(FieldOptions, callback)` 方法建立。該元件接受一個 `FieldOptions` 物件與回呼函式，回呼函式會收到一個 `FieldApi` 物件，該物件提供取得欄位當前值、處理輸入變更與處理模糊事件的方法。

範例：

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

## 欄位狀態 (Field State)

每個欄位都有自身的狀態，包含當前值、驗證狀態、錯誤訊息與其他元資料。您可以透過欄位的 `field.state` 屬性存取其狀態。

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

有三種欄位狀態對於觀察使用者互動非常有用：當使用者點擊/切換至欄位時，欄位會被標記為 _"touched"_；在值被修改前保持 _"pristine"_；值變更後則標記為 _"dirty"_。您可以透過以下所示的 `isTouched`、`isPristine` 與 `isDirty` 標記來檢查這些狀態。

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![欄位狀態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)
