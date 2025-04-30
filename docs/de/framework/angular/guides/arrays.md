---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:55:09.577Z'
id: arrays
title: Arrays
---

TanStack Form unterstützt Arrays als Werte in einem Formular, einschließlich Unterobjekt-Werten innerhalb eines Arrays.

## Grundlegende Verwendung

Um ein Array zu verwenden, können Sie `field.api.state.value` auf einem Array-Wert nutzen:

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

Dies generiert das gemappte JSX jedes Mal, wenn Sie `pushValue` auf `field` ausführen:

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  Person hinzufügen
</button>
```

Schließlich können Sie ein Unterfeld wie folgt verwenden:

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>Name für Person {{ $index }}</div>
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

Hierbei ist `getPeopleName` eine Methode in der Klasse wie folgt:

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> Es ist zwar bedauerlich, dass Sie eine Funktion benötigen, um den Feldnamen zu erhalten, aber dies ist eine Anforderung für unsere strikten TypeScript-Typen.
>
> Wenn wir Folgendes tun würden:
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> Würden wir auf ein TypeScript-Problem stoßen, bei dem `"one" + "two"` als `string` und nicht als der erforderliche `"onetwo"`-Typ behandelt wird.
>
> Darüber hinaus unterstützt Angular zwar Template-Literale im Template, diese dürfen jedoch keine dynamische Interpolation wie unser `$index`-Argument enthalten.

> Es ist möglich, dass wir etwas übersehen haben! Wenn Sie eine bessere Lösung für dieses Problem kennen, [teilen Sie uns dies bitte in unseren GitHub-Diskussionen mit](https://github.com/TanStack/form/discussions).

## Vollständiges Beispiel

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
                    <div>Name für Person {{ $index }}</div>
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
            Person hinzufügen
          </button>
        </ng-container>
      </div>
      <button type="submit" [disabled]="!canSubmit()">
        {{ isSubmitting() ? '...' : 'Absenden' }}
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
