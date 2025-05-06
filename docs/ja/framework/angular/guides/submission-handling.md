---
source-updated-at: '2025-04-27T10:57:16.000Z'
translation-updated-at: '2025-05-06T20:18:58.872Z'
id: submission-handling
title: Submission handling
---

## サブミット処理に追加データを渡す

複数の種類のサブミット動作（例えば別のページに戻るか、フォームに留まるかなど）が必要な場合があります。
これは`onSubmitMeta`プロパティを指定することで実現できます。このメタデータは`onSubmit`関数に渡されます。

> 注: `form.handleSubmit()`がメタデータなしで呼び出された場合、指定されたデフォルト値が使用されます。

```angular-ts
import { Component } from '@angular/core';
import { injectForm } from '@tanstack/angular-form';


type FormMeta = {
  submitAction: 'continue' | 'backToMenu' | null;
};

// form.handleSubmit()を呼び出す際にメタデータは必須ではありません。
// メタデータが渡されない場合に使用するデフォルト値を指定します
const defaultMeta: FormMeta = {
  submitAction: null,
};

@Component({
  selector: 'app-root',
  template: `
    <form (submit)="handleSubmit($event)">
      <button type="submit" (click)="
        handleClick({ submitAction: 'continue' })
      ">送信して続ける</button>
      <button type="submit" (click)="
        handleClick({ submitAction: 'backToMenu' })
      ">送信してメニューに戻る</button>
    </form>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      data: '',
    },
    // サブミット時に期待するメタ値を定義
    onSubmitMeta: defaultMeta,
    onSubmit: async ({ value, meta }) => {
      // handleSubmit経由で渡された値を使用して処理
      console.log(`選択されたアクション - ${meta.submitAction}`, value);
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
  }

  handleClick(meta: FormMeta) {
    // onSubmitMetaで指定されたデフォルトを上書き
    this.form.handleSubmit(meta);
  }
}
```

## スタンダードスキーマでデータを変換する

Tanstack Formはバリデーションのために[スタンダードスキーマサポート](./validation.md)を提供していますが、スキーマの出力データは保持しません。

`onSubmit`関数に渡される値は常に入力データです。スタンダードスキーマの出力データを受け取るには、`onSubmit`関数内でパースします:

```tsx
import { z } from 'zod'
// ...

const schema = z.object({
  age: z.string().transform((age) => Number(age)),
})

// Tanstack Formはスタンダードスキーマの入力型を使用
const defaultValues: z.input<typeof schema> = {
  age: '13',
}

// ...

@Component({
  // ...
})
export class AppComponent {
  form = injectForm({
    defaultValues,
    validators: {
      onChange: schema,
    },
    onSubmit: ({ value }) => {
      const inputAge: string = value.age
      // 変換された値を取得するためにスキーマを通す
      const result = schema.parse(value)
      const outputAge: number = result.age
    },
  })
}
```
