---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:56:51.505Z'
id: form-validation
title: Formularvalidierung
---

# Formular- und Feldvalidierung

Im Kern der Funktionalitäten von TanStack Form liegt das Konzept der Validierung. TanStack Form macht die Validierung hochgradig anpassbar:

- Sie können steuern, wann die Validierung durchgeführt wird (bei Änderung, bei Eingabe, bei Verlassen des Felds, beim Absenden...)
- Validierungsregeln können auf Feldebene oder auf Formularebene definiert werden
- Die Validierung kann synchron oder asynchron erfolgen (z.B. als Ergebnis eines API-Aufrufs)

## Wann wird die Validierung durchgeführt?

Das liegt bei Ihnen! Die `[tanstackField]`-Direktive akzeptiert einige Callbacks als Props wie `onChange` oder `onBlur`. Diese Callbacks erhalten den aktuellen Wert des Felds sowie das `fieldAPI`-Objekt, damit Sie die Validierung durchführen können. Wenn Sie einen Validierungsfehler finden, geben Sie einfach die Fehlermeldung als String zurück, und sie wird in `field.api.state.meta.errors` verfügbar sein.

Hier ein Beispiel:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">Alter:</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined

  // ...
}
```

Im obigen Beispiel wird die Validierung bei jeder Tastatureingabe durchgeführt (`onChange`). Wenn wir stattdessen die Validierung beim Verlassen des Felds (`onBlur`) durchführen wollten, würden wir den Code wie folgt ändern:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onBlur: ageValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">Alter:</label>
      <!-- Wir müssen immer onChange implementieren, damit TanStack Form die Änderungen erhält -->
      <!-- onBlur-Event auf dem Feld abhören -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)='age.api.handleBlur()'
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined

  // ...
}
```

Sie können also steuern, wann die Validierung durchgeführt wird, indem Sie den gewünschten Callback implementieren. Sie können sogar verschiedene Validierungen zu unterschiedlichen Zeitpunkten durchführen:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator,
        onBlur: minimumAgeValidator
      }"
      #age="field"
    >
      <label [for]="age.api.name">Alter:</label>
      <!-- Wir müssen immer onChange implementieren, damit TanStack Form die Änderungen erhält -->
      <!-- onBlur-Event auf dem Feld abhören -->
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (!age.api.state.meta.isValid) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? 'Ungültiger Wert' : undefined)

  // ...
}
```

Im obigen Beispiel validieren wir verschiedene Aspekte desselben Felds zu unterschiedlichen Zeitpunkten (bei jeder Tastatureingabe und beim Verlassen des Felds). Da `field.state.meta.errors` ein Array ist, werden alle relevanten Fehler zu einem bestimmten Zeitpunkt angezeigt. Sie können auch `field.state.meta.errorMap` verwenden, um Fehler basierend auf dem Zeitpunkt der Validierung (onChange, onBlur usw.) abzurufen. Weitere Informationen zur Anzeige von Fehlern finden Sie weiter unten.

## Fehler anzeigen

Sobald Sie Ihre Validierung eingerichtet haben, können Sie die Fehler aus einem Array für die Anzeige in Ihrer Benutzeroberfläche zuordnen:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined

  // ...
}
```

Oder verwenden Sie die Eigenschaft `errorMap`, um auf den spezifischen Fehler zuzugreifen, nach dem Sie suchen:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      @if (age.api.state.meta.errorMap['onChange']) {
        <em role="alert">{{ age.api.state.meta.errorMap['onChange'] }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined

  // ...
}
```

Es ist erwähnenswert, dass unser `errors`-Array und der `errorMap` den von den Validatoren zurückgegebenen Typen entsprechen. Das bedeutet:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{
        onChange: ageValidator
      }"
      #age="field"
    >
      <!-- ... -->
      <!-- errorMap.onChange ist vom Typ `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors ist vom Typ `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">Der Benutzer ist nicht alt genug</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined

  // ...
}
```

## Validierung auf Feldebene vs. auf Formularebene

Wie oben gezeigt, akzeptiert jede `[tanstackField]` ihre eigenen Validierungsregeln über die Callbacks `onChange`, `onBlur` usw. Es ist auch möglich, Validierungsregeln auf Formularebene (im Gegensatz zu Feld für Feld) zu definieren, indem Sie ähnliche Callbacks an die Funktion `injectForm()` übergeben.

Beispiel:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <div>
      <ng-container [tanstackField]="form" name="age" #age="field">
        <!-- ... -->
        @if (formErrorMap().onChange) {
          <div>
            <em
              >Es gab einen Fehler im Formular: {{ formErrorMap().onChange }}</em
            >
          </div>
        }
        <!-- ... -->
      </ng-container>
    </div>
  `,
})
export class AppComponent {
  form = injectForm({
    defaultValues: {
      age: 0,
    },
    onSubmit({ value }) {
      console.log(value)
    },
    validators: {
      // Fügen Sie dem Formular Validatoren hinzu, genauso wie Sie sie einem Feld hinzufügen würden
      onChange({ value }) {
        if (value.age < 13) {
          return 'Du musst mindestens 13 Jahre alt sein, um dich anzumelden'
        }
        return undefined
      },
    },
  })

  // Abonnieren Sie die Fehlerzuordnung des Formulars, damit Aktualisierungen gerendert werden
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

### Festlegen von Fehlern auf Feldebene aus den Validatoren des Formulars

Sie können Fehler in den Feldern aus den Validatoren des Formulars festlegen. Ein häufiger Anwendungsfall hierfür ist die Validierung aller Felder beim Absenden durch Aufruf eines einzelnen API-Endpunkts im `onSubmitAsync`-Validator des Formulars.

```angular-ts
@Component({
  selector: 'app-root',
  imports: [TanStackField],
  template: `
    <form (submit)="handleSubmit($event)">
      <div>
        <ng-container
          [tanstackField]="form"
          name="age"
          #ageField="field"
        >
          <label [for]="ageField.api.name">Alter:</label>
          <input
            type="number"
            [name]="ageField.api.name"
            [value]="ageField.api.state.value"
            (blur)="ageField.api.handleBlur()"
            (input)="ageField.api.handleChange($any($event).target.valueAsNumber)"
          />
          @if (ageField.api.state.meta.errors.length > 0) {
            <em role="alert">{{ ageField.api.state.meta.errors.join(', ') }}</em>
          }
        </ng-container>
      </div>
      <button type="submit">Absenden</button>
    </form>
  `,
})

export class AppComponent {
  form = injectForm({
    defaultValues: {
      age: 0,
      socials: [],
      details: {
        email: '',
      },
    },
    validators: {
      onSubmitAsync: async ({ value }) => {
        // Validieren Sie den Wert auf dem Server
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Ungültige Daten', // Der Schlüssel `form` ist optional
            fields: {
              age: 'Du musst mindestens 13 Jahre alt sein, um dich anzumelden',
              // Fehler in verschachtelten Feldern mit dem Feldnamen festlegen
              'socials[0].url': 'Die angegebene URL existiert nicht',
              'details.email': 'Eine E-Mail-Adresse ist erforderlich',
            },
          };
        }

        return null;
      },
    },
  });

  handleSubmit(event: SubmitEvent) {
    event.preventDefault();
    event.stopPropagation();
    this.form.handleSubmit();
  }
}
```

> Es ist erwähnenswert, dass, wenn Sie eine Formularvalidierungsfunktion haben, die einen Fehler zurückgibt, dieser Fehler durch die feld-spezifische Validierung überschrieben werden kann.
>
> Das bedeutet:
>
> ```angular-ts
> @Component({
>   selector: 'app-root',
>   standalone: true,
>   imports: [TanStackField],
>   template: `
>       <div>
>         <ng-container
>          [tanstackField]="form"
>         name="age"
>        #ageField="field"
>       [validators]="{
>        onChange: fieldValidator
>     }"
>  >
>   <input type="number" [value]="ageField.api.state.value"
>   (input)="ageField.api.handleChange($any($event).target.valueAsNumber)"
>   />
>    @if (ageField.api.state.meta.errors.length > 0) {
>       <em role="alert">{{ ageField.api.state.meta.errors.join(', ') }}</em>
>     }
>   </ng-container>
> </div>
> `,
> })
> export class AppComponent {
>   form = injectForm({
>     defaultValues: {
>       age: 0,
>     },
>     validators: {
>       onChange: ({ value }) => {
>         return {
>           fields: {
>             age: value.age < 12 ? 'Zu jung!' : undefined,
>           },
>         };
>       },
>     },
>   });
>
>   fieldValidator: FieldValidateFn<any, any, number> = ({ value }) =>
>     value % 2 === 0 ? 'Muss ungerade sein!' : undefined;
> }
>
> ```
>
> Wird nur `'Muss ungerade sein!'` anzeigen, selbst wenn der 'Zu jung!'-Fehler von der Formular-Validierung zurückgegeben wird.

## Asynchrone funktionale Validierung

Während wir vermuten, dass die meisten Validierungen synchron sein werden, gibt es viele Fälle, in denen ein Netzwerkaufruf oder eine andere asynchrone Operation nützlich wäre.

Dafür haben wir dedizierte Methoden wie `onChangeAsync`, `onBlurAsync` usw., die zur Validierung verwendet werden können:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onChangeAsync: ageValidator }"
      #age="field"
    >
      <label [for]="age.api.name">Nachname:</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type="number"
        (input)="age.api.handleChange($any($event).target.valueAsNumber)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    await new Promise((resolve) => setTimeout(resolve, 1000))
    return value < 13 ? 'Du musst mindestens 13 Jahre alt sein, um ein Konto zu erstellen' : undefined
  }

  // ...
}
```

Synchrone und asynchrone Validierungen können nebeneinander existieren. Beispielsweise ist es möglich, sowohl `onBlur` als auch `onBlurAsync` für dasselbe Feld zu definieren:

```angular-ts
@Component({
  selector: 'app-root',
  standalone: true,
  imports: [TanStackField],
  template: `
    <ng-container
      [tanstackField]="form"
      name="age"
      [validators]="{ onBlur: ensureAge13, onBlurAsync: ensureOlderAge }"
      #age="field"
    >
      <label [for]="age.api.name">Nachname:</label>
      <input
        [id]="age.api.name"
        [name]="age.api.name"
        [value]="age.api.state.value"
        type='number'
        (blur)="age.api.handleBlur()"
        (input)="age.api.handleChange($any($event).target.value)"
      />
      @if (age.api.state.meta.errors) {
        <em role="alert">{{ age.api.state.meta.errors.join(', ') }}</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ensureAge13: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Du musst mindestens 13 Jahre alt
```
