---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:26:48.166Z'
id: arrays
title: 配列
---

TanStack Formは、フォーム内の値として配列をサポートしており、配列内のサブオブジェクト値も扱うことができます。

## 基本的な使い方

配列を使用するには、配列値に対して`field.api.state.value`を使用します:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="people" #people="field">
      <div>
        @for (_ of people.api.state.value; track $index) {
          <!-- ... -->
        }
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      people: [] as Array<{ name: string; age: number }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

これは`field`に対して`pushValue`を実行するたびにマップされたJSXを生成します:

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  人物を追加
</button>
```

最後に、サブフィールドを次のように使用できます:

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>人物 {{ $index }} の名前</div>
      <input
        [value]="person.api.state.value"
        (input)="
          person.api.handleChange($any($event).target.value)
        "
      />
    </label>
  </div>
</ng-container>
```

ここで`getPeopleName`はクラスのメソッドで、次のようになります:

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> フィールド名を取得するために関数を使用する必要があるのは残念ですが、これは厳密なTypeScriptの型が機能するための要件です。
>
> 例えば、次のようにした場合:
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> `"one" + "two"`が`string`型ではなく、必要な`"onetwo"`型にならないというTypeScriptの問題が発生します。
>
> さらに、Angularはテンプレート内でテンプレートリテラルをサポートしていますが、`$index`引数のような動的な補間を含めることはできません。

> 私たちが見落としている可能性もあります！この問題に対するより良い解決策を思いついた場合は、[GitHubディスカッションでお知らせください](https://github.com/TanStack/form/discussions)。

## 完全な例

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <form (submit)="handleSubmit($event)">
      <div>
        <ng-container [tanstackField]="form" name="people" #people="field">
          <div>
            @for (_ of people.api.state.value; track $index) {
              <ng-container
                [tanstackField]="form"
                [name]="getPeopleName($index)"
                #person="field"
              >
                <div>
                  <label>
                    <div>人物 {{ $index }} の名前</div>
                    <input
                      [value]="person.api.state.value"
                      (input)="
                        person.api.handleChange($any($event).target.value)
                      "
                    />
                  </label>
                </div>
              </ng-container>
            }
          </div>
          <button (click)="people.api.pushValue(defaultPerson)" type="button">
            人物を追加
          </button>
        </ng-container>
      </div>
      <button type="submit" [disabled]="!canSubmit()">
        {{ isSubmitting() ? '...' : '送信' }}
      </button>
    </form>
  `,
})
export class AppComponent {
  defaultPerson = { name: '', age: 0 }

  form = injectForm({
    defaultValues: {
      people: [] as Array<{ name: string; age: number }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })


  getPeopleName = (idx: number) => `people[${idx}].name` as const;

  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)

  handleSubmit(event: SubmitEvent) {
    event.preventDefault()
    event.stopPropagation()
    this.form.handleSubmit()
  }
}
```
