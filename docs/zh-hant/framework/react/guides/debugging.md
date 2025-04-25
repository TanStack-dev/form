---
source-updated-at: '2025-03-28T15:34:36.000Z'
translation-updated-at: '2025-04-25T20:39:21.215Z'
id: debugging
title: 除錯
---
以下是您可能會在控制台中看到的常見錯誤列表及解決方法。

## 將非受控輸入變更為受控輸入

如果您在控制台中看到此錯誤：

```
警告：一個元件正在將非受控輸入變更為受控。這可能是由於值從 undefined 變更為已定義值所導致，此情況不應發生。請在元件生命週期內決定使用受控或非受控輸入元素。更多資訊：https://reactjs.org/link/controlled-components
```

這可能是因為您忘記在 `useForm` Hook 或 `form.Field` 元件中使用 `defaultValues`。此錯誤發生的原因是輸入在表單值初始化之前就被渲染，因此在文字輸入時值會從 `undefined` 變更為 `""`。

## 欄位值的類型為 `unknown`

如果您使用 `form.Field` 並在檢查 `field.state.value` 時發現欄位值的類型為 `unknown`，這可能是因為您的表單類型過大，導致無法安全評估。

這通常表示您應該將表單拆分成較小的表單，或為表單使用更具體的類型。

解決此問題的一種方法是使用 TypeScript 的 `as` 關鍵字來轉換 `field.state.value`：

```tsx
const value = field.state.value as string
```

## `類型實例化過深且可能無限`

如果您在執行 `tsc` 時在控制台中看到此錯誤：

```
類型實例化過深且可能無限
```

這表示您遇到了我們在類型定義中未發現的錯誤。雖然我們已盡力確保類型的準確性，但在某些邊緣情況下，TypeScript 可能會因類型的複雜性而無法處理。

請[在 GitHub 上向我們回報此問題](https://github.com/TanStack/form/issues)，以便我們進行修復。請確保提供最小重現步驟，以便我們協助您除錯。

> 請注意，此錯誤是 TypeScript 錯誤而非執行時錯誤。這表示您的程式碼在用戶端仍會如預期般執行。
