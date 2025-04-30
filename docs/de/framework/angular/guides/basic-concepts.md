---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:55:44.735Z'
id: basic-concepts
title: Grundkonzepte
---

Diese Seite führt die grundlegenden Konzepte und Begriffe ein, die in der `@tanstack/angular-form`-Bibliothek verwendet werden. Das Vertrautmachen mit diesen Konzepten hilft Ihnen, die Bibliothek besser zu verstehen und damit zu arbeiten.

## Formular-Instanz (Form Instance)

Eine Formular-Instanz ist ein Objekt, das ein einzelnes Formular repräsentiert und Methoden sowie Eigenschaften für die Arbeit mit dem Formular bereitstellt. Sie erstellen eine Formular-Instanz mit der Funktion `injectForm`. Der Hook akzeptiert ein Objekt mit einer `onSubmit`-Funktion, die beim Absenden des Formulars aufgerufen wird.

```typescript
const form = injectForm({
  onSubmit: async ({ value }) => {
    // Mit den Formulardaten etwas tun
    console.log(value)
  },
})
```

## Feld (Field)

Ein Feld (Field) repräsentiert ein einzelnes Formular-Eingabeelement, wie z.B. ein Textfeld oder eine Checkbox. Felder werden mit der Direktive `tanstackField` erstellt. Die Direktive akzeptiert eine `name`-Eigenschaft, die mit einem Schlüssel in den Standardwerten des Formulars übereinstimmen sollte. Sie stellt auch eine `field`-Instanz der internen Direktivenlogik bereit, die über eine Template-Variable genutzt werden sollte, um auf die Interna des Feldes zuzugreifen.

Beispiel:

```html
<ng-container [tanstackField]="form" name="firstName" #firstName="field">
  <input
    [value]="firstName.api.state.value"
    (blur)="firstName.api.handleBlur()"
    (input)="firstName.api.handleChange($any($event).target.value)"
  />
</ng-container>
```

## Feldzustand (Field State)

Jedes Feld hat seinen eigenen Zustand, der seinen aktuellen Wert, Validierungsstatus, Fehlermeldungen und andere Metadaten enthält. Sie können den Zustand eines Feldes über die Eigenschaft `fieldApi.state` abrufen.

Beispiel:

```tsx
const {
  value,
  meta: { errors, isValidating },
} = field.state
```

Es gibt drei Feldzustände, die sehr nützlich sind, um zu sehen, wie der Benutzer mit einem Feld interagiert. Ein Feld ist _"berührt" (touched)_, wenn der Benutzer hineinklickt/-tabuliert, _"unverändert" (pristine)_, bis der Benutzer den Wert ändert, und _"verändert" (dirty)_, nachdem der Wert geändert wurde. Sie können diese Zustände über die Flags `isTouched`, `isPristine` und `isDirty` prüfen, wie unten gezeigt.

```tsx
const { isTouched, isPristine, isDirty } = field.state.meta
```

![Feldzustände](https://raw.githubusercontent.com/TanStack/form/main/docs/assets/field-states.png)

## Feld-API (Field API)

Die Feld-API ist ein Objekt, das über die Eigenschaft `tanstackField.api` beim Erstellen eines Feldes zugänglich ist. Sie bietet Methoden für die Arbeit mit dem Zustand des Feldes.

Beispiel:

```angular-html
<input
  [value]="fieldName.api.state.value"
  (blur)="fieldName.api.handleBlur()"
  (input)="fieldName.api.handleChange($any($event).target.value)"
/>
```

## Validierung (Validation)

`@tanstack/angular-form` bietet sowohl synchrone als auch asynchrone Validierung von Haus aus. Validierungsfunktionen können der `tanstackField`-Direktive über die `validators`-Eigenschaft übergeben werden.

Beispiel:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="firstName" #firstName="field">
      <input
        [value]="firstName.api.state.value"
        (blur)="firstName.api.handleBlur()"
        (input)="firstName.api.handleChange($any($event).target.value)"
      />
    </ng-container>
  `,
})
export class AppComponent {
  firstNameValidator: FieldValidateFn<any, any, string, any> = ({
                                                                       value,
                                                                     }) =>
    !value
      ? 'Ein Vorname ist erforderlich'
      : value.length < 3
        ? 'Der Vorname muss mindestens 3 Zeichen lang sein'
        : undefined

  firstNameAsyncValidator: FieldValidateAsyncFn<any, string, any> =
    async ({ value }) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return value.includes('error') && '"error" ist im Vornamen nicht erlaubt'
    }

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      console.log(value)
    },
  })
}
```

## Validierung mit Standard-Schema-Bibliotheken (Validation with Standard Schema Libraries)

Zusätzlich zu manuell erstellten Validierungsoptionen unterstützen wir auch die [Standard-Schema](https://github.com/standard-schema/standard-schema)-Spezifikation.

Sie können ein Schema mit einer der Bibliotheken definieren, die die Spezifikation implementieren, und es einem Formular- oder Feld-Validator übergeben.

Unterstützte Bibliotheken umfassen:

- [Zod](https://zod.dev/)
- [Valibot](https://valibot.dev/)
- [ArkType](https://arktype.io/)

Beispiel:

```angular-ts
import { z } from 'zod'

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="firstName"
      [validators]="{
        onChange: z.string().min(3, 'Der Vorname muss mindestens 3 Zeichen lang sein'),
        onChangeAsyncDebounceMs: 500,
        onChangeAsync: firstNameAsyncValidator
      }"
      #firstName="field"
    >
      <!-- ... -->
    </ng-container>
  `,
})
export class AppComponent {
  firstNameAsyncValidator = z.string().refine(
    async (value) => {
      await new Promise((resolve) => setTimeout(resolve, 1000))
      return !value.includes('error')
    },
    {
      message: '"error" ist im Vornamen nicht erlaubt',
    },
  )

  form = injectForm({
    defaultValues: {
      firstName: '',
    },
    onSubmit({ value }) {
      // Mit den Formulardaten etwas tun
      console.log(value)
    },
  })

  z = z
}
```

## Reaktivität (Reactivity)

`@tanstack/angular-form` bietet eine Möglichkeit, Änderungen am Formular- und Feldzustand über `injectStore(this.form, selector)` zu abonnieren.

Beispiel:

```typescript
import { injectForm, injectStore } from '@tanstack/angular-form'

@Component(/*...*/)
class AppComponent {
  form = injectForm(/*...*/)
  canSubmit = injectStore(this.form, (state) => state.canSubmit)
  isSubmitting = injectStore(this.form, (state) => state.isSubmitting)
}
```

## Listener (Listeners)

`@tanstack/angular-form` ermöglicht es Ihnen, auf bestimmte Trigger zu reagieren und diese zu "überwachen", um Nebenwirkungen auszulösen.

Beispiel:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="country"
      [listeners]="{
        onChange: onCountryChange
      }"
      #country="field"
    ></ng-container>
  `,
})

...

onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`Land geändert zu: ${value}, Provinz wird zurückgesetzt`)
    this.form.setFieldValue('province', '')
  }
```

Weitere Informationen finden Sie unter [Listener](./listeners.md)

## Array-Felder (Array Fields)

Array-Felder ermöglichen es Ihnen, eine Liste von Werten innerhalb eines Formulars zu verwalten, wie z.B. eine Liste von Hobbys. Sie können ein Array-Feld mit der `tanstackField`-Direktive erstellen.

Bei der Arbeit mit Array-Feldern können Sie die Methoden `pushValue`, `removeValue` und `swapValues` der Felder verwenden, um Werte im Array hinzuzufügen, zu entfernen oder zu vertauschen.

Beispiel:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container [tanstackField]="form" name="hobbies" #hobbies="field">
      <div>
        Hobbys
        <div>
          @if (!hobbies.api.state.value.length) {
            Keine Hobbys gefunden
          }
          @for (_ of hobbies.api.state.value; track $index) {
            <div>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyName($index)"
                #hobbyName="field"
              >
                <div>
                  <label [for]="hobbyName.api.name">Name:</label>
                  <input
                    [id]="hobbyName.api.name"
                    [name]="hobbyName.api.name"
                    [value]="hobbyName.api.state.value"
                    (blur)="hobbyName.api.handleBlur()"
                    (input)="
                      hobbyName.api.handleChange($any($event).target.value)
                    "
                  />
                  <button
                    type="button"
                    (click)="hobbies.api.removeValue($index)"
                  >
                    X
                  </button>
                </div>
              </ng-container>
              <ng-container
                [tanstackField]="form"
                [name]="getHobbyDesc($index)"
                #hobbyDesc="field"
              >
                <div>
                  <label [for]="hobbyDesc.api.name">Beschreibung:</label>
                  <input
                    [id]="hobbyDesc.api.name"
                    [name]="hobbyDesc.api.name"
                    [value]="hobbyDesc.api.state.value"
                    (blur)="hobbyDesc.api.handleBlur()"
                    (input)="
                      hobbyDesc.api.handleChange($any($event).target.value)
                    "
                  />
                </div>
              </ng-container>
            </div>
          }
        </div>
        <button type="button" (click)="hobbies.api.pushValue(defaultHobby)">
          Hobby hinzufügen
        </button>
      </div>
    </ng-container>
  `,
})
export class AppComponent {
  defaultHobby = {
    name: '',
    description: '',
    yearsOfExperience: 0,
  }

  getHobbyName = (idx: number) => `hobbies[${idx}].name` as const;
  getHobbyDesc = (idx: number) => `hobbies[${idx}].description` as const;

  form = injectForm({
    defaultValues: {
      hobbies: [] as Array<{
        name: string
        description: string
        yearsOfExperience: number
      }>,
    },
    onSubmit({ value }) {
      alert(JSON.stringify(value))
    },
  })
}
```

Dies sind die grundlegenden Konzepte und Begriffe, die in der `@tanstack/angular-form`-Bibliothek verwendet werden. Das Verständnis dieser Konzepte hilft Ihnen, effektiver mit der Bibliothek zu arbeiten und komplexe Formulare mit Leichtigkeit zu erstellen.
