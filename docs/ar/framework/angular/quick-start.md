---
source-updated-at: '2024-07-19T07:45:40.000Z'
translation-updated-at: '2025-04-30T22:04:22.576Z'
id: quick-start
title: بدء سريع
---

الحد الأدنى للبدء مع TanStack Form هو إنشاء نموذج وإضافة حقل. ضع في الاعتبار أن هذا المثال لا يتضمن أي تحقق من صحة البيانات أو معالجة الأخطاء... حتى الآن.

```angular-ts
import { Component } from '@angular/core'
import { bootstrapApplication } from '@angular/platform-browser'
import { TanStackField, injectForm } from '@tanstack/angular-form'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <form (submit)="handleSubmit($event)">
      <div>
        <ng-container
          [tanstackField]="form"
          name="fullName"
          #fullName="field"
        >
          <label [for]="fullName.api.name">First Name:</label>
          <input
            [name]="fullName.api.name"
            [value]="fullName.api.state.value"
            (blur)="fullName.api.handleBlur()"
            (input)="fullName.api.handleChange($any($event).target.value)"
          />
        </ng-container>
      </div>
      <button type="submit">Submit</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      fullName: '',
    },
    onSubmit({ value }) {
      // Do something with form data
      console.log(value)
    },
  })

  handleSubmit(event: SubmitEvent) {
    event.preventDefault()
    event.stopPropagation()
    this.form.handleSubmit()
  }
}

bootstrapApplication(AppComponent).catch((err) => console.error(err))
```

من هنا، ستكون جاهزًا لاستكشاف جميع الميزات الأخرى في TanStack Form!
