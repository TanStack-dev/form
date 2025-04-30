---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:44:25.672Z'
id: form-validation
title: Валидация формы
---
Основой функциональности TanStack Form является концепция валидации. TanStack Form делает валидацию максимально настраиваемой:

- Вы можете контролировать, когда выполнять валидацию (при изменении, при вводе, при потере фокуса, при отправке...)
- Правила валидации можно определять на уровне поля или на уровне формы
- Валидация может быть синхронной или асинхронной (например, в результате вызова API)

## Когда выполняется валидация?

Это зависит от вас! Директива `[tanstackField]` принимает некоторые колбэки в качестве пропсов, такие как `onChange` или `onBlur`. Эти колбэки получают текущее значение поля, а также объект `fieldAPI`, чтобы вы могли выполнить валидацию. Если обнаружена ошибка валидации, просто верните сообщение об ошибке в виде строки, и оно будет доступно в `field.api.state.meta.errors`.

Вот пример:

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
      <label [for]="age.api.name">Age:</label>
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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

В примере выше валидация выполняется при каждом нажатии клавиши (`onChange`). Если бы мы хотели, чтобы валидация выполнялась при потере фокуса полем, мы бы изменили код следующим образом:

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
      <label [for]="age.api.name">Age:</label>
      <!-- onChange всегда нужно реализовывать, чтобы TanStack Form получал изменения -->
      <!-- Слушаем событие onBlur на поле -->
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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

Таким образом, вы можете контролировать, когда выполняется валидация, реализуя нужный колбэк. Вы даже можете выполнять разные проверки в разное время:

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
      <label [for]="age.api.name">Age:</label>
      <!-- onChange всегда нужно реализовывать, чтобы TanStack Form получал изменения -->
      <!-- Слушаем событие onBlur на поле -->
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
    value < 13 ? 'You must be 13 to make an account' : undefined

  minimumAgeValidator: FieldValidateFn<any, any, any, any, number> = ({
    value,
  }) => (value < 0 ? 'Invalid value' : undefined)

  // ...
}
```

В примере выше мы проверяем разные условия для одного поля в разное время (при каждом нажатии клавиши и при потере фокуса). Поскольку `field.state.meta.errors` является массивом, отображаются все актуальные ошибки на данный момент. Вы также можете использовать `field.state.meta.errorMap` для получения ошибок в зависимости от _того, когда_ была выполнена валидация (onChange, onBlur и т.д.). Подробнее о выводе ошибок ниже.

## Отображение ошибок

После настройки валидации вы можете отображать ошибки из массива в вашем интерфейсе:

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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

Или использовать свойство `errorMap` для доступа к конкретной ошибке:

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
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

Стоит отметить, что массив `errors` и `errorMap` соответствуют типам, возвращаемым валидаторами. Это означает, что:

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
      <!-- errorMap.onChange имеет тип `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors имеет тип `Array<{isOldEnough: false} | undefined>` -->
      @if (!age.api.state.meta.errorMap['onChange']?.isOldEnough) {
        <em role="alert">The user is not old enough</em>
      }
    </ng-container>
  `,
})
export class AppComponent {
  ageValidator: FieldValidateFn<any, any, any, any, number> = ({ value }) =>
    value < 13 ? 'You must be 13 to make an account' : undefined

  // ...
}
```

## Валидация на уровне поля vs на уровне формы

Как показано выше, каждый `[tanstackField]` принимает свои собственные правила валидации через колбэки `onChange`, `onBlur` и т.д. Также можно определить правила валидации на уровне формы (в отличие от поля) путем передачи аналогичных колбэков в функцию `injectForm()`.

Пример:

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
              >There was an error on the form: {{ formErrorMap().onChange }}</em
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
      // Добавляем валидаторы к форме так же, как и к полю
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // Подписываемся на errorMap формы, чтобы обновления отображались
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

### Установка ошибок на уровне полей из валидаторов формы

Вы можете устанавливать ошибки на полях из валидаторов формы. Один из распространенных случаев использования — валидация всех полей при отправке путем вызова одного API-эндпоинта в валидаторе `onSubmitAsync` формы.

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
          <label [for]="ageField.api.name">Age:</label>
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
      <button type="submit">Submit</button>
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
        // Валидируем значение на сервере
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // Ключ `form` опционален
            fields: {
              age: 'Must be 13 or older to sign',
              // Устанавливаем ошибки на вложенные поля с помощью имени поля
              'socials[0].url': 'The provided URL does not exist',
              'details.email': 'An email is required',
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

> Стоит отметить, что если у вас есть функция валидации формы, которая возвращает ошибку, эта ошибка может быть перезаписана валидацией конкретного поля.
>
> Это означает, что:
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
>             age: value.age < 12 ? 'Too young!' : undefined,
>           },
>         };
>       },
>     },
>   });
>
>   fieldValidator: FieldValidateFn<any, any, number> = ({ value }) =>
>     value % 2 === 0 ? 'Must be odd!' : undefined;
> }
>
> ```
>
> Будет показывать только `'Must be odd!'`, даже если ошибка 'Too young!' возвращается валидацией на уровне формы.

## Асинхронная функциональная валидация

Хотя мы предполагаем, что большинство валидаций будут синхронными, есть много случаев, когда вызов сети или другая асинхронная операция может быть полезна для валидации.

Для этого у нас есть специальные методы `onChangeAsync`, `onBlurAsync` и другие, которые можно использовать для валидации:

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
      <label [for]="age.api.name">Last Name:</label>
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
    return value < 13 ? 'You must be 13 to make an account' : undefined
  }

  // ...
}
```

Синхронные и асинхронные валидации могут сосуществовать. Например, можно определить и `onBlur`, и `onBlurAsync` для одного поля:

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
      <label [for]="age.api.name">Last Name:</label>
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
    value < 13 ? 'You must be at least 13' : undefined

  ensureOlderAge: FieldValidateAsyncFn<any, string, number> = async ({
    value,
  }) => {
    const currentAge = await fetchCurrentAgeOnProfile()
    return value < currentAge ? 'You can only increase the age' : undefined
  }

  // ...
}
```

Сначала выполняется синхронный метод валидации (`onBlur`), а асинхронный метод (`onBlurAsync`) выполняется только в случае успешного выполнения синхронного (`onBlur`). Чтобы изменить это поведение, установите опцию `asyncAlways` в `true`, и асинхронный метод будет выполняться независимо от результата синхронного.

### Встроенный дебаунсинг

Хотя асинхронные вызовы — это правильный способ валидации против базы данных, выполнение сетевого запроса при каждом нажатии клавиши — хороший способ DDOS-ата
