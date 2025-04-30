---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T21:35:29.222Z'
id: react-native
title: React Native
---

## Motivation

La plupart des frameworks web ne proposent pas de solution complète pour la gestion des formulaires, obligeant les développeurs à créer leurs propres implémentations personnalisées ou à s'appuyer sur des bibliothèques moins performantes. Cela entraîne souvent un manque de cohérence, des performances médiocres et un temps de développement accru. TanStack Form vise à résoudre ces problèmes en fournissant une solution tout-en-un pour la gestion des formulaires, à la fois puissante et facile à utiliser.

Avec TanStack Form, les développeurs peuvent relever les défis courants liés aux formulaires, tels que :

- La liaison de données réactive et la gestion d'état
- La validation complexe et la gestion des erreurs
- L'accessibilité et la conception responsive
- L'internationalisation et la localisation
- La compatibilité multiplateforme et le style personnalisé

En offrant une solution complète à ces défis, TanStack Form permet aux développeurs de créer des formulaires robustes et conviviaux avec facilité.

TanStack Form est "headless" et devrait prendre en charge React Native sans configuration supplémentaire.

Voici un exemple :

```tsx
<form.Field
  name="age"
  validators={{
    onChange: (val) =>
      val < 13 ? 'You must be 13 to make an account' : undefined,
  }}
>
  {(field) => (
    <>
      <Text>Age:</Text>
      <TextInput value={field.state.value} onChangeText={field.handleChange} />
      {!field.state.meta.isValid && (
        <Text>{field.state.meta.errors.join(', ')}</Text>
      )}
    </>
  )}
</form.Field>
```
