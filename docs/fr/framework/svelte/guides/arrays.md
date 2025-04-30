---
source-updated-at: '2025-04-24T18:50:42.000Z'
translation-updated-at: '2025-04-30T21:38:09.175Z'
id: arrays
title: Tableaux
---

# Tableaux (Arrays)

TanStack Form prend en charge les tableaux comme valeurs dans un formulaire, y compris les sous-objets à l'intérieur d'un tableau.

## Utilisation de base

Pour utiliser un tableau, vous pouvez utiliser `field.state.value` sur une valeur de tableau en conjonction avec les [`blocs each`](https://svelte.dev/docs/svelte/each) :

```svelte
<script>
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      people: [],
    },
  }))
</script>

<form.Field name="people" mode="array">
  {#snippet children(field)}
    {#each field.state.value as person, i}
      <!-- ... -->
    {/each}
  {/snippet}
</form.Field>
```

Cela régénérera la liste à chaque fois que vous utiliserez `pushValue` sur `field` :

```svelte
<button onclick={() => field.pushValue({ name: '', age: 0 })} type="button">
  Ajouter une personne
</button>
```

Enfin, vous pouvez utiliser un sous-champ comme ceci :

```svelte
<form.Field name={`people[${i}].name`}>
  {#snippet children(subField)}
    <input
      value={subField.state.value}
      oninput={(e) => {
        subField.handleChange(e.currentTarget.value)
      }}
    />
  {/snippet}
</form.Field>
```

## Exemple complet

```svelte
<script lang="ts">
  import { createForm } from '@tanstack/svelte-form'

  const form = createForm(() => ({
    defaultValues: {
      people: [] as Array<{ age: number; name: string }>,
    },
    onSubmit: ({ value }) => alert(JSON.stringify(value)),
  }))
</script>

<form
  id="form"
  onsubmit={(e) => {
    e.preventDefault()
    e.stopPropagation()
    form.handleSubmit()
  }}
>
  <form.Field name="people">
    {#snippet children(field)}
      <div>
        {#each field.state.value as person, i}
          <form.Field name={`people[${i}].name`}>
            {#snippet children(subField)}
              <div>
                <label>
                  <div>Nom pour la personne {i}</div>
                  <input
                    value={person.name}
                    oninput={(e: Event) => {
                      const target = e.target as HTMLInputElement
                      subField.handleChange(target.value)
                    }}
                  />
                </label>
              </div>
            {/snippet}
          </form.Field>
        {/each}

        <button
          onclick={() => field.pushValue({ name: '', age: 0 })}
          type="button"
        >
          Ajouter une personne
        </button>
      </div>
    {/snippet}
  </form.Field>

  <button type="submit"> Soumettre </button>
</form>
```
