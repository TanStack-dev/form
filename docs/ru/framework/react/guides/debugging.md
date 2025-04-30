---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-30T21:59:43.225Z'
id: debugging
title: Отладка
---

Вот список распространённых ошибок, которые вы можете увидеть в консоли, и способы их исправления.

## Изменение неуправляемого (uncontrolled) ввода на управляемый (controlled)

Если в консоли появляется следующая ошибка:

```
Warning: A component is changing an uncontrolled input to be controlled. This is likely caused by the value changing from undefined to a defined value, which should not happen. Decide between using a controlled or uncontrolled input element for the lifetime of the component. More info: https://reactjs.org/link/controlled-components
```

Скорее всего, вы забыли указать `defaultValues` в хуке `useForm` или при использовании компонента `form.Field`. Это происходит потому, что ввод отображается до инициализации значения формы, и его значение меняется с `undefined` на `""` при вводе текста.

## Значение поля имеет тип `unknown`

Если вы используете `form.Field` и при проверке значения `field.state.value` видите, что тип поля — `unknown`, вероятно, тип вашей формы слишком сложен для безопасного определения.

Обычно это означает, что форму следует разбить на более мелкие формы или использовать более конкретный тип.

Временное решение — приведение типа `field.state.value` с помощью ключевого слова `as` в TypeScript:

```tsx
const value = field.state.value as string
```

## `Type instantiation is excessively deep and possibly infinite`

Если при запуске `tsc` в консоли появляется ошибка:

```
Type instantiation is excessively deep and possibly infinite
```

Вы столкнулись с ошибкой, которую мы не учли в наших определениях типов. Хотя мы постарались сделать типы максимально точными, в некоторых крайних случаях TypeScript не справляется с их сложностью.

Пожалуйста, [сообщите об этой проблеме на GitHub](https://github.com/TanStack/form/issues), чтобы мы могли её исправить. Обязательно приложите минимальный воспроизводимый пример, чтобы мы могли помочь с отладкой.

> Помните, что это ошибка TypeScript, а не ошибка времени выполнения. Это означает, что ваш код будет работать на стороне пользователя так, как ожидается.
