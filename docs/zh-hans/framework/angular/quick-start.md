---
source-updated-at: '2024-07-19T07:45:40.000Z'
translation-updated-at: '2025-04-12T04:08:25.901Z'
id: quick-start
title: 快速开始
---
## 快速入门

使用 TanStack Form 的最基本步骤是创建一个表单并添加字段。请注意，以下示例尚未包含任何验证或错误处理功能。

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
      // 对表单数据进行处理
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

完成这个基础示例后，您就可以开始探索 TanStack Form 的所有其他功能了！
