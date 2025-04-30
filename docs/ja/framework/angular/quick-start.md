---
source-updated-at: '2024-07-19T07:45:40.000Z'
translation-updated-at: '2025-04-30T21:24:31.722Z'
id: quick-start
title: クイックスタート
---

# クイックスタート

TanStack Formを始めるための最低限の手順は、フォームを作成してフィールドを追加することです。この例にはまだバリデーションやエラーハンドリングは含まれていないことに注意してください。

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

ここから、TanStack Formの他のすべての機能を探索する準備が整いました！

## モチベーション

ほとんどのウェブフレームワークはフォーム処理に関する包括的なソリューションを提供しておらず、開発者は独自のカスタム実装を作成するか、機能が限られたライブラリに頼らざるを得ません。これにより、一貫性の欠如、パフォーマンスの低下、開発時間の増加といった問題がしばしば発生します。TanStack Formは、これらの課題を解決するために、強力で使いやすいフォーム管理のオールインワンソリューションを提供することを目指しています。

TanStack Formを使用すれば、開発者は以下のような一般的なフォーム関連の課題に対処できます：

- リアクティブなデータバインディングとステート管理
- 複雑なバリデーションとエラーハンドリング
- アクセシビリティとレスポンシブデザイン
- 国際化 (i18n) とローカライゼーション
- クロスプラットフォーム互換性とカスタムスタイリング

これらの課題に対する完全なソリューションを提供することで、TanStack Formは開発者が堅牢でユーザーフレンドリーなフォームを簡単に構築できるように支援します。
