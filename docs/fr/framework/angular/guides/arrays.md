---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:37:00.290Z'
id: arrays
title: Tableaux
---

# Tableaux (Arrays)

TanStack Form prend en charge les tableaux comme valeurs dans un formulaire, y compris les sous-objets à l'intérieur d'un tableau.

## Utilisation de base

Pour utiliser un tableau, vous pouvez utiliser `field.api.state.value` sur une valeur de tableau :

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

Cela générera le JSX mappé à chaque fois que vous exécutez `pushValue` sur `field` :

```angular-html
<button (click)="people.api.pushValue(defaultPerson)" type="button">
  Ajouter une personne
</button>
```

Enfin, vous pouvez utiliser un sous-champ comme ceci :

```angular-html
 <ng-container
  [tanstackField]="form"
  [name]="getPeopleName($index)"
  #person="field"
>
  <div>
    <label>
      <div>Nom pour la personne {{ $index }}</div>
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

Où `getPeopleName` est une méthode dans la classe comme ceci :

```typescript
export class AppComponent {
  getPeopleName = (idx: number) => `people[${idx}].name` as const

  // ...
}
```

> Bien qu'il soit regrettable que vous ayez besoin d'utiliser une fonction pour obtenir le nom du champ, c'est une exigence pour le fonctionnement de nos types TypeScript stricts.
>
> Voyez, si nous faisions ce qui suit :
>
> ```angular-html
> <ng-container [tanstackField]="form" [name]="'people[' + $index + '].name'"></ng-container>
> ```
>
> Nous rencontrerions un problème TypeScript où `"one" + "two"` est de type `string` plutôt que le type requis `"onetwo"`
>
> De plus, bien qu'Angular prenne en charge les littéraux de modèle dans le template, ils ne peuvent pas contenir d'interpolation dynamique en leur sein, comme notre argument `$index`.

> Il est possible que nous ayons manqué quelque chose ! Si vous avez une meilleure solution à ce problème, [contactez-nous via les discussions GitHub](https://github.com/TanStack/form/discussions).

## Exemple complet

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
                    <div>Nom pour la personne {{ $index }}</div>
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
            Ajouter une personne
          </button>
        </ng-container>
      </div>
      <button type="submit" [disabled]="!canSubmit()">
        {{ isSubmitting() ? '...' : 'Soumettre' }}
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
