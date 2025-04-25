---
source-updated-at: '2025-04-02T16:32:51.000Z'
translation-updated-at: '2025-04-25T20:55:00.403Z'
id: FieldComponent
title: 類型 / FieldComponent
---
# 類型別名: FieldComponent()\<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta\>

```ts
type FieldComponent<TParentData, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta> = <TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>(props) => CreateComponentPublicInstanceWithMixins<FieldComponentBoundProps<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync>, {}, {}, {}, {}, ComponentOptionsMixin, ComponentOptionsMixin, EmitsOptions, PublicProps, {}, false, {}, SlotsType<{
  default: {
     field: FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>;
     state: FieldApi<TParentData, TName, TData, TOnMount, TOnChange, TOnChangeAsync, TOnBlur, TOnBlurAsync, TOnSubmit, TOnSubmitAsync, TFormOnMount, TFormOnChange, TFormOnChangeAsync, TFormOnBlur, TFormOnBlurAsync, TFormOnSubmit, TFormOnSubmitAsync, TFormOnServer, TParentSubmitMeta>["state"];
    };
}>>;
```

定義於: [packages/vue-form/src/useField.tsx:24](https://github.com/TanStack/form/blob/main/packages/vue-form/src/useField.tsx#L24)

## 類型參數

• **TParentData**

• **TFormOnMount** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChange** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnChangeAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnBlur** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnBlurAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnSubmit** *繼承自* `undefined` \| `FormValidateOrFn`\<`TParentData`\>

• **TFormOnSubmitAsync** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TFormOnServer** *繼承自* `undefined` \| `FormAsyncValidateOrFn`\<`TParentData`\>

• **TParentSubmitMeta**

## 參數

### props

[`FieldComponentBoundProps`](fieldcomponentboundprops.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`\> & `EmitsToProps`\<`EmitsOptions`\> & `PublicProps`

## 返回值

`CreateComponentPublicInstanceWithMixins`\<[`FieldComponentBoundProps`](fieldcomponentboundprops.md)\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`\>, \{\}, \{\}, \{\}, \{\}, `ComponentOptionsMixin`, `ComponentOptionsMixin`, `EmitsOptions`, `PublicProps`, \{\}, `false`, \{\}, `SlotsType`\<\{
  `default`: \{
     `field`: `FieldApi`\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TParentSubmitMeta`\>;
     `state`: `FieldApi`\<`TParentData`, `TName`, `TData`, `TOnMount`, `TOnChange`, `TOnChangeAsync`, `TOnBlur`, `TOnBlurAsync`, `TOnSubmit`, `TOnSubmitAsync`, `TFormOnMount`, `TFormOnChange`, `TFormOnChangeAsync`, `TFormOnBlur`, `TFormOnBlurAsync`, `TFormOnSubmit`, `TFormOnSubmitAsync`, `TFormOnServer`, `TParentSubmitMeta`\>\[`"state"`\];
    \};
 \}\>\>
