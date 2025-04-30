---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:38:42.988Z'
id: form-validation
title: Validation de formulaire
---

Au cœur des fonctionnalités de TanStack Form se trouve le concept de validation. TanStack Form rend la validation hautement personnalisable :

- Vous pouvez contrôler quand effectuer la validation (à chaque changement, à la saisie, lors de la perte de focus, à la soumission...)
- Les règles de validation peuvent être définies au niveau du champ ou au niveau du formulaire
- La validation peut être synchrone ou asynchrone (par exemple, suite à un appel API)

## Quand la validation est-elle effectuée ?

C'est à vous de décider ! La directive `[tanstackField]` accepte certaines fonctions de rappel (callbacks) comme props, telles que `onChange` ou `onBlur`. Ces callbacks reçoivent la valeur actuelle du champ ainsi que l'objet fieldAPI, vous permettant ainsi d'effectuer la validation. Si vous détectez une erreur de validation, il suffit de retourner le message d'erreur sous forme de chaîne de caractères, et celui-ci sera disponible dans `field.api.state.meta.errors`.

Voici un exemple :

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
      <label [for]="age.api.name">Âge :</label>
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
    value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined

  // ...
}
```

Dans l'exemple ci-dessus, la validation est effectuée à chaque frappe (`onChange`). Si, à la place, nous voulions que la validation soit effectuée lors de la perte de focus (blur), nous modifierions le code comme suit :

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
      <label [for]="age.api.name">Âge :</label>
      <!-- Nous devons toujours implémenter onChange pour que TanStack Form reçoive les modifications -->
      <!-- Écouter l'événement onBlur sur le champ -->
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
    value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined

  // ...
}
```

Ainsi, vous pouvez contrôler quand la validation est effectuée en implémentant le callback souhaité. Vous pouvez même effectuer différentes validations à différents moments :

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
      <label [for]="age.api.name">Âge :</label>
      <!-- Nous devons toujours implémenter onChange pour que TanStack Form reçoive les modifications -->
      <!-- Écouter l'événement onBlur sur le champ -->
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
    value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? 'Valeur invalide' : undefined)

  // ...
}
```

Dans l'exemple ci-dessus, nous validons différentes choses sur le même champ à différents moments (à chaque frappe et lors de la perte de focus). Comme `field.state.meta.errors` est un tableau, toutes les erreurs pertinentes à un moment donné sont affichées. Vous pouvez également utiliser `field.state.meta.errorMap` pour obtenir les erreurs en fonction de _quand_ la validation a été effectuée (onChange, onBlur, etc.). Plus d'informations sur l'affichage des erreurs ci-dessous.

## Affichage des erreurs

Une fois votre validation en place, vous pouvez mapper les erreurs depuis un tableau pour les afficher dans votre interface :

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
    value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined

  // ...
}
```

Ou utiliser la propriété `errorMap` pour accéder à une erreur spécifique :

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
    value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined

  // ...
}
```

Il est important de noter que notre tableau `errors` et le `errorMap` correspondent aux types retournés par les validateurs. Cela signifie que :

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
      <!-- errorMap.onChange est de type `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors est de type `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">L'utilisateur n'est pas assez âgé</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined

  // ...
}
```

## Validation au niveau du champ vs au niveau du formulaire

Comme montré ci-dessus, chaque `[tanstackField]` accepte ses propres règles de validation via les callbacks `onChange`, `onBlur`, etc. Il est également possible de définir des règles de validation au niveau du formulaire (plutôt que champ par champ) en passant des callbacks similaires à la fonction `injectForm()`.

Exemple :

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
              >Une erreur est survenue dans le formulaire : {{ formErrorMap().onChange }}</em
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
      // Ajouter des validateurs au formulaire de la même manière que pour un champ
      onChange({ value }) {
        if (value.age < 13) {
          return 'Vous devez avoir 13 ans ou plus pour vous inscrire'
        }
        return undefined
      },
    },
  })

  // S'abonner à la map d'erreurs du formulaire pour que les mises à jour soient rendues
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

### Définir des erreurs au niveau des champs depuis les validateurs du formulaire

Vous pouvez définir des erreurs sur les champs depuis les validateurs du formulaire. Un cas d'usage courant est de valider tous les champs à la soumission en appelant un seul endpoint API dans le validateur `onSubmitAsync` du formulaire.

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
          <label [for]="ageField.api.name">Âge :</label>
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
      <button type="submit">Soumettre</button>
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
        // Valider la valeur côté serveur
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Données invalides', // La clé `form` est optionnelle
            fields: {
              age: 'Vous devez avoir 13 ans ou plus pour vous inscrire',
              // Définir des erreurs sur des champs imbriqués avec le nom du champ
              'socials[0].url': "L'URL fournie n'existe pas",
              'details.email': 'Un email est requis',
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

> Il est important de noter que si vous avez une fonction de validation de formulaire qui retourne une erreur, cette erreur peut être écrasée par la validation spécifique au champ.
>
> Cela signifie que :
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
>             age: value.age < 12 ? 'Trop jeune !' : undefined,
>           },
>         };
>       },
>     },
>   });
>
>   fieldValidator: FieldValidateFn<any, any, number> = ({ value }) =>
>     value % 2 === 0 ? 'Doit être impair !' : undefined;
> }
>
> ```
>
> Affichera uniquement `'Doit être impair !'` même si l'erreur 'Trop jeune !' est retournée par la validation au niveau du formulaire.

## Validation fonctionnelle asynchrone

Bien que nous supposions que la plupart des validations seront synchrones, il existe de nombreux cas où un appel réseau ou une autre opération asynchrone serait utile pour valider.

Pour cela, nous avons des méthodes dédiées `onChangeAsync`, `onBlurAsync`, etc. qui peuvent être utilisées pour valider :

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
      <label [for]="age.api.name">Âge :</label>
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
    return value < 13 ? 'Vous devez avoir 13 ans pour créer un compte' : undefined
  }

  // ...
}
```

Les validations synchrones et asynchrones peuvent coexister. Par exemple, il est possible de définir à la fois `onBlur` et `onBlurAsync` sur le même champ :

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
      <label [for]="age.api.name">Âge :</label>
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
    value < 13 ? 'Vous devez avoir au moins 13 ans' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? 'Vous ne pouvez qu\'augmenter l\'âge' : undefined
  }

  // ...
}
```

La méthode de validation synchrone (`onBlur`) est exécutée en premier, et la méthode asynchrone (`onBlurAsync`) n'est exécutée que si la première réussit. Pour modifier ce comportement, définissez l'
