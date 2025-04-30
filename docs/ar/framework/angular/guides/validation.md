---
source-updated-at: '2025-04-29T10:21:58.000Z'
translation-updated-at: '2025-04-30T22:08:22.298Z'
id: form-validation
title: التحقق من صحة النموذج
---

في صلب وظائف TanStack Form تكمن فكرة التحقق (validation). يجعل TanStack Form التحقق قابلاً للتخصيص بدرجة كبيرة:

- يمكنك التحكم في وقت إجراء التحقق (عند التغيير، عند الإدخال، عند فقدان التركيز، عند الإرسال...)
- يمكن تعريف قواعد التحقق على مستوى الحقل أو على مستوى النموذج
- يمكن أن يكون التحقق متزامنًا أو غير متزامن (على سبيل المثال، نتيجة لاستدعاء API)

## متى يتم إجراء التحقق؟

الأمر يعود إليك! توجيه `[tanstackField]` يقبل بعض الدوال ردود فعل كـ props مثل `onChange` أو `onBlur`. يتم تمرير القيمة الحالية للحقل بالإضافة إلى كائن fieldAPI إلى هذه الدوال، مما يتيح لك إجراء التحقق. إذا وجدت خطأ في التحقق، ما عليك سوى إرجاع رسالة الخطأ كسلسلة نصية وسيتم تخزينها في `field.api.state.meta.errors`.

إليك مثالًا:

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

في المثال أعلاه، يتم التحقق عند كل ضغطة مفتاح (`onChange`). إذا أردنا بدلاً من ذلك أن يتم التحقق عند فقدان التركيز على الحقل، سنغير الكود كما يلي:

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
      <!-- نحتاج دائمًا إلى تنفيذ onChange حتى يتلقى TanStack Form التغييرات -->
      <!-- الاستماع إلى حدث onBlur على الحقل -->
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

لذا يمكنك التحكم في وقت إجراء التحقق من خلال تنفيذ رد الفعل المطلوب. يمكنك حتى تنفيذ أجزاء مختلفة من التحقق في أوقات مختلفة:

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
      <!-- نحتاج دائمًا إلى تنفيذ onChange حتى يتلقى TanStack Form التغييرات -->
      <!-- الاستماع إلى حدث onBlur على الحقل -->
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

في المثال أعلاه، نقوم بالتحقق من أشياء مختلفة على نفس الحقل في أوقات مختلفة (عند كل ضغطة مفتاح وعند فقدان التركيز على الحقل). نظرًا لأن `field.state.meta.errors` هي مصفوفة، يتم عرض جميع الأخطاء ذات الصلة في وقت معين. يمكنك أيضًا استخدام `field.state.meta.errorMap` للحصول على الأخطاء بناءً على _وقت_ إجراء التحقق (onChange، onBlur إلخ...). المزيد من المعلومات حول عرض الأخطاء أدناه.

## عرض الأخطاء

بمجرد وضع التحقق الخاص بك، يمكنك تعيين الأخطاء من مصفوفة ليتم عرضها في واجهة المستخدم الخاصة بك:

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

أو استخدم خاصية `errorMap` للوصول إلى الخطأ المحدد الذي تبحث عنه:

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

من الجدير بالذكر أن مصفوفة `errors` و `errorMap` تتطابق مع الأنواع التي يتم إرجاعها بواسطة أدوات التحقق. هذا يعني أن:

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
      <!-- errorMap.onChange هو من النوع `{isOldEnough: false} | undefined` -->
	  <!-- meta.errors هو من النوع `Array<{isOldEnough: false} | undefined>` -->
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

## التحقق على مستوى الحقل مقابل مستوى النموذج

كما هو موضح أعلاه، يقبل كل `[tanstackField]` قواعد التحقق الخاصة به عبر دوال ردود الفعل مثل `onChange`، `onBlur` إلخ... من الممكن أيضًا تعريف قواعد التحقق على مستوى النموذج (بدلاً من حقل بحقل) عن طريق تمرير دوال ردود فعل مماثلة إلى دالة `injectForm()`.

مثال:

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
      // أضف أدوات التحقق إلى النموذج بنفس الطريقة التي تضيفها إلى حقل
      onChange({ value }) {
        if (value.age < 13) {
          return 'Must be 13 or older to sign'
        }
        return undefined
      },
    },
  })

  // اشترك في خريطة الأخطاء للنموذج بحيث سيتم عرض التحديثات عليها
  formErrorMap = injectStore(this.form, (state) => state.errorMap)
}
```

### تعيين أخطاء على مستوى الحقل من أدوات التحقق الخاصة بالنموذج

يمكنك تعيين أخطاء على الحقول من أدوات التحقق الخاصة بالنموذج. أحد حالات الاستخدام الشائعة لهذا هو التحقق من جميع الحقول عند الإرسال عن طريق استدعاء نقطة نهاية API واحدة في أداة التحقق `onSubmitAsync` للنموذج.

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
        // تحقق من القيمة على الخادم
        const hasErrors = await verifyDataOnServer(value)
        if (hasErrors) {
          return {
            form: 'Invalid data', // مفتاح `form` اختياري
            fields: {
              age: 'Must be 13 or older to sign',
              // تعيين أخطاء على الحقول المتداخلة مع اسم الحقل
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

> شيء جدير بالذكر هو أنه إذا كان لديك دالة تحقق للنموذج تُرجع خطأً، فقد يتم استبدال هذا الخطأ بتحقق محدد للحقل.
>
> هذا يعني أن:
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
> سيظهر فقط `'Must be odd!` حتى إذا تم إرجاع خطأ 'Too young!' بواسطة تحقق مستوى النموذج.

## التحقق الوظيفي غير المتزامن

بينما نعتقد أن معظم عمليات التحقق ستكون متزامنة، هناك العديد من الحالات التي يكون فيها استدعاء شبكة أو عملية غير متزامنة أخرى مفيدة للتحقق منها.

للقيام بذلك، لدينا طرق مخصصة مثل `onChangeAsync`، `onBlurAsync`، وغيرها التي يمكن استخدامها للتحقق:

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

يمكن أن يتعايش التحقق المتزامن وغير المتزامن. على سبيل المثال، من الممكن تعريف كل من `onBlur` و `onBlurAsync` على نفس الحقل:

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

يتم تشغيل طريقة التحقق المتزامنة (`onBlur`) أولاً وتتم تشغيل الطريقة غير المتزامنة (`onBlurAsync`)
