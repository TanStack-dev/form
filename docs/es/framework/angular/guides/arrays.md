---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:49:24.388Z'
id: arrays
title: Arreglos
---

TanStack Form admite arrays como valores en un formulario, incluyendo valores de subobjetos dentro de un array.

## Uso básico

Para usar un array, puedes utilizar `field.api.state.value` en un valor de array:

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

Esto generará el JSX mapeado cada vez que ejecutes `pushValue` en `field`:

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  Añadir persona
</button>
```

Finalmente, puedes usar un subcampo de la siguiente manera:

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>Nombre para la persona {{ $index }}</div>
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

Donde `getPeopleName` es un método en la clase como este:

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> Si bien es desafortunado que necesites usar una función para obtener el nombre del campo, es un requisito para cómo funcionan nuestros tipos estrictos de TypeScript.
>
> Observa que si hiciéramos lo siguiente:
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> Nos encontraríamos con un problema de TypeScript donde `"one" + "two"` es `string` en lugar del tipo requerido `"onetwo"`.
>
> Además, aunque Angular admite literales de plantilla en la plantilla, no pueden contener interpolación dinámica dentro de ellos, como nuestro argumento `$index`.

> ¡Es posible que hayamos pasado algo por alto! Si se te ocurre una mejor solución para este problema, [déjanos un mensaje en nuestras discusiones de GitHub](https://github.com/TanStack/form/discussions).

## Ejemplo completo

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
                    <div>Nombre para la persona {{ $index }}</div>
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
            Añadir persona
          </button>
        </ng-container>
      </div>
      <button type="submit" [disabled]="!canSubmit()">
        {{ isSubmitting() ? '...' : 'Enviar' }}
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
