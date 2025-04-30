---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:27:18.853Z'
id: basic-concepts
title: 基本概念
---

# 基本概念と用語

このページでは、`@tanstack/angular-form`ライブラリで使用される基本概念と用語を紹介します。これらの概念を理解することで、ライブラリをより効果的に活用できるようになります。

## フォームインスタンス (Form Instance)

フォームインスタンスは、個々のフォームを表現するオブジェクトで、フォームを操作するためのメソッドとプロパティを提供します。フォームインスタンスは`injectForm`関数を使用して作成します。この関数は`onSubmit`関数を含むオブジェクトを受け取り、フォームが送信されると呼び出されます。

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // フォームデータで何か処理を行う
    console.log(value)
  },
})
```

## フィールド (Field)

フィールドは、テキスト入力やチェックボックスなどの単一のフォーム入力要素を表します。フィールドは`tanstackField`ディレクティブを使用して作成します。このディレクティブは、フォームのデフォルト値のキーと一致する必要があるnameプロパティを受け取り、フィールドの内部状態にアクセスするために使用する`field`という名前のディレクティブインスタンスを公開します。

例:

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## フィールド状態 (Field State)

各フィールドには、現在の値、検証ステータス、エラーメッセージ、その他のメタデータを含む独自の状態があります。フィールドの状態には`fieldApi.state`プロパティを使用してアクセスできます。

例:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

ユーザーがフィールドとどのようにやり取りするかを確認するのに役立つ3つのフィールド状態があります。フィールドは、ユーザーがクリック/タブでフォーカスすると「touched」、値が変更されるまでは「pristine」、値が変更された後は「dirty」とマークされます。これらの状態は、以下のように`isTouched`、`isPristine`、`isDirty`フラグで確認できます。

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![フィールド状態](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## フィールドAPI (Field API)

フィールドAPIは、フィールドを作成する際に`tanstackField.api`プロパティでアクセスできるオブジェクトで、フィールドの状態を操作するためのメソッドを提供します。

例:

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## バリデーション (Validation)

`@tanstack/angular-form`は、同期バリデーションと非同期バリデーションの両方を標準で提供しています。バリデーション関数は、`validators`プロパティを使用して`tanstackField`ディレクティブに渡すことができます。

例:

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
      ? '名は必須です'
      : value.length < 3
        ? '名は3文字以上必要です'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '名に"error"を含めることはできません'
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

## 標準スキーマライブラリを使ったバリデーション (Validation with Standard Schema Libraries)

手動で作成したバリデーションオプションに加えて、[Standard Schema](https://github.com/standard-schema/standard-schema)仕様もサポートしています。

この仕様を実装した任意のライブラリを使用してスキーマを定義し、フォームまたはフィールドバリデータに渡すことができます。

サポートされているライブラリ:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

例:

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
        onChange: z.string().min(3, '名は3文字以上必要です'),
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
      message: "名に'error'を含めることはできません",
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // フォームデータで何か処理を行う
      console.log(value)
    },
  })

  z = z
}
```

## リアクティビティ (Reactivity)

`@tanstack/angular-form`は、`injectStore(this.form, selector)`を使用してフォームとフィールドの状態変更を購読する方法を提供します。

例:

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## リスナー (Listeners)

`@tanstack/angular-form`では、特定のトリガーに反応し、それらを「リッスン」して副作用をディスパッチすることができます。

例:

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
    console.log(`国が${value}に変更されました、州をリセットします`)
    this.form.setFieldValue('province', '')
  }
```

詳細は[リスナー](./listeners.md)をご覧ください

## 配列フィールド (Array Fields)

配列フィールドを使用すると、趣味のリストなどのフォーム内の値のリストを管理できます。配列フィールドは`tanstackField`ディレクティブを使用して作成します。

配列フィールドを操作する際は、フィールドの`pushValue`、`removeValue`、`swapValues`メソッドを使用して、配列内の値を追加、削除、交換できます。

例:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        趣味
        <div>
          @if (!hobbies.api.state.value.length) {
            趣味が見つかりません
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">名前:</label>
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
                  <label [for]="hobbyDesc.api.name">説明:</label>
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
          趣味を追加
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

これらが`@tanstack/angular-form`ライブラリで使用される基本概念と用語です。これらの概念を理解することで、ライブラリをより効果的に使用し、複雑なフォームを簡単に作成できるようになります。
