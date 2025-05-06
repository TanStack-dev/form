---
source-updated-at: '2024-11-25T20:59:59.000Z'
translation-updated-at: '2025-05-06T20:24:50.949Z'
id: listeners
title: Side effects for event triggers
---

Pour les situations où vous souhaitez "affecter" ou "réagir" à des déclencheurs, il existe l'API d'écoute (listener API). Par exemple, si vous, en tant que développeur, souhaitez réinitialiser un champ de formulaire suite à la modification d'un autre champ, vous utiliserez l'API d'écoute.

Imaginez le flux utilisateur suivant :

- L'utilisateur sélectionne un pays dans une liste déroulante.
- L'utilisateur sélectionne ensuite une province dans une autre liste déroulante.
- L'utilisateur change le pays sélectionné pour un autre.

Dans cet exemple, lorsque l'utilisateur change le pays, la province sélectionnée doit être réinitialisée car elle n'est plus valide. Avec l'API d'écoute, nous pouvons nous abonner à l'événement onChange et déclencher une réinitialisation du champ "province" lorsque l'écouteur est activé.

Les événements pouvant être "écoutés" sont :

- onChange
- onBlur
- onMount
- onSubmit

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

    <ng-container
      [tanstackField]="form"
      name="province"
      #province="field"
    ></ng-container>
  `,
})

export class AppComponent {
  form = injectForm({
    defaultValues: {
      country: '',
      province: '',
    },
  })

  onCountryChange: FieldListenerFn<any, any, any, any, string> = ({
    value,
  }) => {
    console.log(`Country changed to: ${value}, resetting province`)
    this.form.setFieldValue('province', '')
  }
}
```
