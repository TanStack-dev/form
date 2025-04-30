---
source-updated-at: '2025-03-04T11:48:41.000Z'
translation-updated-at: '2025-04-30T21:35:09.944Z'
id: linked-fields
title: Champs liés
---

Vous pourriez avoir besoin de lier deux champs de formulaire ensemble ; lorsque l'un est validé suite à la modification de la valeur d'un autre champ.  
Un cas d'usage courant est lorsque vous avez à la fois un champ `password` et un champ `confirm_password`,  
où vous souhaitez que `confirm_password` affiche une erreur si sa valeur ne correspond pas à celle de `password`,  
peu importe quel champ a déclenché la modification.

Imaginez le flux utilisateur suivant :

- L'utilisateur modifie le champ de confirmation de mot de passe.
- L'utilisateur modifie le champ de mot de passe principal.

Dans cet exemple, le formulaire contiendra toujours des erreurs,  
car la validation du champ "confirm password" n'a pas été relancée pour marquer la valeur comme acceptée.

Pour résoudre ce problème, nous devons nous assurer que la validation du champ "confirm password" est relancée lorsque le champ `password` est mis à jour.  
Pour ce faire, vous pouvez ajouter une propriété `onChangeListenTo` au champ `confirm_password`.

```tsx
function App() {
  const form = useForm({
    defaultValues: {
      password: '',
      confirm_password: '',
    },
    // ...
  })

  return (
    <div>
      <form.Field name="password">
        {(field) => (
          <label>
            <div>Password</div>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </label>
        )}
      </form.Field>
      <form.Field
        name="confirm_password"
        validators={{
          onChangeListenTo: ['password'],
          onChange: ({ value, fieldApi }) => {
            if (value !== fieldApi.form.getFieldValue('password')) {
              return 'Passwords do not match'
            }
            return undefined
          },
        }}
      >
        {(field) => (
          <div>
            <label>
              <div>Confirm Password</div>
              <input
                value={field.state.value}
                onChange={(e) => field.handleChange(e.target.value)}
              />
            </label>
            {field.state.meta.errors.map((err) => (
              <div key={err}>{err}</div>
            ))}
          </div>
        )}
      </form.Field>
    </div>
  )
}
```

Cela fonctionne également avec la propriété `onBlurListenTo`, qui relancera la validation lorsque le champ perd le focus.
