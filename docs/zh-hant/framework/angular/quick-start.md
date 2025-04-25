---
source-updated-at: '2024-07-19T07:45:40.000Z'
translation-updated-at: '2025-04-25T20:31:42.999Z'
id: quick-start
title: 快速開始
---
## 快速開始

使用 TanStack Form 最基本的入門步驟是建立一個表單並添加欄位。請注意，這個範例尚未包含任何驗證或錯誤處理功能...（目前僅展示基礎用法）

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

從這裡開始，您就可以開始探索 TanStack Form 的所有其他功能了！
