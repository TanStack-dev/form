---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T22:06:46.725Z'
id: arrays
title: المصفوفات
---

يدعم TanStack Form المصفوفات كقيم في النموذج، بما في ذلك قيم الكائنات الفرعية داخل المصفوفة.

## الاستخدام الأساسي

لاستخدام مصفوفة، يمكنك استخدام `field.api.state.value` على قيمة المصفوفة:

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

سيؤدي هذا إلى إنشاء JSX المعين في كل مرة تقوم فيها باستدعاء `pushValue` على `field`:

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  Add person
</button>
```

أخيرًا، يمكنك استخدام حقل فرعي كما يلي:

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>Name for person {{ $index }}</div>
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

حيث أن `getPeopleName` هي دالة في الفئة كما يلي:

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> بينما من المؤسف أنك تحتاج إلى استخدام دالة للحصول على اسم الحقل، إلا أنه شرط مطلوب لكيفية عمل أنواع TypeScript الصارمة لدينا.
>
> انظر، إذا قمنا بما يلي:
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> سنواجه مشكلة في TypeScript حيث أن `"one" + "two"` تعطي `string` بدلاً من النوع المطلوب `"onetwo"`
>
> علاوة على ذلك، بينما يدعم Angular القوالب الحرفية في القالب، لا يمكن أن تحتوي على إقحام ديناميكي بداخلها، مثل وسيطة `$index` الخاصة بنا.

> من الممكن أننا فاتنا شيئًا! إذا كان بإمكانك التفكير في حل أفضل لهذه المشكلة، [أرسل لنا رسالة في مناقشات GitHub الخاصة بنا](https://github.com/TanStack/form/discussions).

## مثال كامل

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
                    <div>Name for person {{ $index }}</div>
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
            Add person
          </button>
        </ng-container>
      </div>
      <button type="submit" [disabled]="!canSubmit()">
        {{ isSubmitting() ? '...' : 'Submit' }}
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
