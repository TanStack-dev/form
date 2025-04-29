---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-29T23:12:00.853Z'
id: arrays
title: 陣列
---

TanStack Form 支援將陣列 (arrays) 作為表單值，包含陣列中的子物件值。

## 基本用法

要使用陣列，可以在陣列值上使用 `field.api.state.value`：

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

每次在 `field` 上執行 `pushValue` 時，都會生成對應的 JSX：

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  新增人員
</button>
```

最後，可以像這樣使用子欄位 (subfield)：

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>人員 {{ $index }} 的名稱</div>
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

其中 `getPeopleName` 是類別中的方法，如下所示：

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> 雖然需要使用函式來取得欄位名稱有點不便，但這是為了符合我們嚴格的 TypeScript 型別系統的要求。
>
> 舉例來說，如果我們這樣做：
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> 會遇到 TypeScript 的問題，因為 `"one" + "two"` 的型別是 `string` 而非所需的 `"onetwo"` 字面值型別 (literal type)
>
> 此外，雖然 Angular 支援模板字面值 (template literals)，但不能在其中動態插入變數，例如我們的 `$index` 參數。

> 有可能我們遺漏了某些解決方案！如果您有更好的解決方法，[歡迎在 GitHub 討論區告訴我們](https://github.com/TanStack/form/discussions)。

## 完整範例

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
                    <div>人員 {{ $index }} 的名稱</div>
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
            新增人員
          </button>
        </ng-container>
      </div>
      <button type="submit" [disabled]="!canSubmit()">
        {{ isSubmitting() ? '...' : '提交' }}
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
