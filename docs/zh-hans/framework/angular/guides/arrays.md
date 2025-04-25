---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-12T04:10:43.653Z'
id: arrays
title: 数组
---
TanStack Form 支持将数组作为表单值使用，包括数组中的子对象值。

## 基本用法

要在数组值上使用数组功能，可以通过 `field.api.state.value` 访问：

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

每次在 `field` 上调用 `pushValue` 时都会生成映射后的 JSX：

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  添加人员
</button>
```

子字段可以这样使用：

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>人员 {{ $index }} 的姓名</div>
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

其中 `getPeopleName` 是组件类中的方法：

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> 虽然需要通过函数获取字段名有些不便，但这是为了满足我们严格的 TypeScript 类型要求。
>
> 例如，如果直接这样写：
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> 会遇到 TypeScript 问题，因为 `"one" + "two"` 会被识别为 `string` 类型而非所需的 `"onetwo"` 字面量类型
>
> 此外，虽然 Angular 模板支持模板字面量，但不能包含动态插值（如 `$index` 参数）

> 可能我们遗漏了更好的解决方案！如果您有改进建议，[欢迎在 GitHub 讨论区留言](https://github.com/TanStack/form/discussions)。

## 完整示例

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
                    <div>人员 {{ $index }} 的姓名</div>
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
            添加人员
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
