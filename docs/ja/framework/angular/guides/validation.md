---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:28:31.119Z'
id: form-validation
title: フォームバリデーション
---

TanStack Formの機能の中核にあるのがバリデーションの概念です。TanStack Formではバリデーションを高度にカスタマイズ可能に設計しています:

- バリデーションを実行するタイミングを制御可能（変更時、入力時、フォーカスアウト時、送信時など）
- バリデーションルールはフィールドレベルまたはフォームレベルで定義可能
- 同期バリデーションと非同期バリデーション（API呼び出し結果など）の両方をサポート

## バリデーションの実行タイミング

実行タイミングは開発者が自由に選択できます。`[tanstackField]`ディレクティブは`onChange`や`onBlur`などのコールバックをpropsとして受け取ります。これらのコールバックにはフィールドの現在値とfieldAPIオブジェクトが渡されるので、バリデーションを実行できます。バリデーションエラーが見つかった場合、単にエラーメッセージを文字列で返すと、`field.api.state.meta.errors`で利用可能になります。

以下は実装例です:

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
      <label [for]="age.api.name">年齢:</label>
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
    value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined

  // ...
}
```

上記の例では、バリデーションは各キーストローク時（`onChange`）に実行されます。代わりにフィールドのフォーカスが外れた時（`onBlur`）にバリデーションを行いたい場合は、以下のようにコードを変更します:

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
      <label [for]="age.api.name">年齢:</label>
      <!-- TanStack Formが変更を受け取るため、常にonChangeを実装する必要があります -->
      <!-- フィールドのonBlurイベントをリッスン -->
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
    value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined

  // ...
}
```

このように、実装するコールバックによってバリデーションの実行タイミングを制御できます。異なるタイミングで異なるバリデーションを実行することも可能です:

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
      <label [for]="age.api.name">年齢:</label>
      <!-- TanStack Formが変更を受け取るため、常にonChangeを実装する必要があります -->
      <!-- フィールドのonBlurイベントをリッスン -->
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
    value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? '無効な値です' : undefined)

  // ...
}
```

上記の例では、同じフィールドに対して異なるタイミング（各キーストローク時とフォーカスアウト時）で異なるバリデーションを実行しています。`field.state.meta.errors`は配列なので、特定の時点で関連する全てのエラーが表示されます。また、`field.state.meta.errorMap`を使用して、バリデーションが実行されたタイミング（onChange、onBlurなど）に基づいてエラーを取得することもできます。エラー表示に関する詳細は後述します。

## エラーの表示

バリデーションを設定したら、エラーを配列からマッピングしてUIに表示できます:

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
    value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined

  // ...
}
```

または、`errorMap`プロパティを使用して特定のエラーにアクセスすることも可能です:

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
    value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined

  // ...
}
```

`errors`配列と`errorMap`はバリデーターが返す型と一致することに注意してください。つまり:

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
      <!-- errorMap.onChangeの型は `{isOldEnough: false} | undefined` -->
	  <!-- meta.errorsの型は `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">ユーザーは十分な年齢に達していません</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined

  // ...
}
```

## フィールドレベル vs フォームレベルのバリデーション

上記で示したように、各`[tanstackField]`は`onChange`、`onBlur`などのコールバックを通じて独自のバリデーションルールを受け入れます。また、`injectForm()`関数に類似のコールバックを渡すことで、フィールドごとではなくフォームレベルでバリデーションルールを定義することも可能です。

例:

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
              >フォームにエラーがあります: {{ formErrorMap().onChange }}</em
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
      // フィールドに追加するのと同じ方法でフォームにバリデーターを追加
      onChange({ value }) {
        if (value.age < 13) {
          return 'サインアップには13歳以上である必要があります'
        }
        return undefined
      },
    },
  })

  // フォームのエラーマップを購読し、更新時にレンダリングされるようにする
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

### フォームバリデーターからフィールドレベルのエラーを設定

フォームのバリデーターからフィールドにエラーを設定できます。この機能の一般的な使用例は、フォームの`onSubmitAsync`バリデーターで単一のAPIエンドポイントを呼び出して全てのフィールドをバリデーションすることです。

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
          <label [for]="ageField.api.name">年齢:</label>
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
      <button type="submit">送信</button>
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
        // サーバーで値を検証
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: '無効なデータ', // `form`キーはオプション
            fields: {
              age: 'サインアップには13歳以上である必要があります',
              // フィールド名でネストしたフィールドにエラーを設定
              'socials[0].url': '提供されたURLは存在しません',
              'details.email': 'メールアドレスは必須です',
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

> 注意点として、フォームバリデーション関数がエラーを返した場合、そのエラーはフィールド固有のバリデーションによって上書きされる可能性があります。
>
> つまり:
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
>             age: value.age < 12 ? '年齢が低すぎます！' : undefined,
>           },
>         };
>       },
>     },
>   });
>
>   fieldValidator: FieldValidateFn<any, any, number> = ({ value }) =>
>     value % 2 === 0 ? '奇数でなければなりません！' : undefined;
> }
>
> ```
>
> この場合、フォームレベルのバリデーションが「年齢が低すぎます！」というエラーを返しても、「奇数でなければなりません！」というエラーのみが表示されます。

## 非同期関数バリデーション

多くのバリデーションは同期処理になると予想されますが、ネットワーク呼び出しやその他の非同期操作を使用して検証する必要があるケースも多く存在します。

このため、`onChangeAsync`、`onBlurAsync`などの専用メソッドを用意しています:

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
      <label [for]="age.api.name">年齢:</label>
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
    return value < 13 ? 'アカウント作成には13歳以上である必要があります' : undefined
  }

  // ...
}
```

同期バリデーションと非同期バリデーションは共存可能です。例えば、同じフィールドに`onBlur`と`onBlurAsync`の両方を定義できます:

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
      <label [for]="age.api.name">年齢:</label>
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
    value < 13 ? '13歳以上である必要があります' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? '年齢は増加のみ可能です' : undefined
  }

  // ...
```
