# Copy Address Value When Checkbox Is Already Selected

## Problem

In a SurveyJS form, I use a `copyvalue` trigger to copy the `Address` value into `BillingAddress` when the `CopyAddress` checkbox is selected. This works when toggling `CopyAddress`, but if `CopyAddress` is already `true` and `Address` changes, `BillingAddress` is not updated.

```json
{
  "elements": [
    { "type": "comment", "name": "Address", "title": "Address", "isRequired": true },
    { "type": "boolean", "name": "CopyAddress", "title": "Use the address for billing" },
    { "type": "comment", "name": "BillingAddress", "title": "Billing Address" }
  ],
  "triggers": [
    {
      "type": "copyvalue",
      "fromName": "Address",
      "setToName": "BillingAddress",
      "expression": "{CopyAddress} = true"
    }
  ],
  "textUpdateMode": "onTyping"
}
```

## Solution

In SurveyJS, the `copyvalue` trigger copies the value from `fromName` to `setToName` when the `expression` evaluates to `true`. However, the trigger only re-runs when variables in the `expression` (e.g., `CopyAddress`) change. In this case, Address isn't part of the expression, so changing it doesn't re-run the trigger. The following options ensure `BillingAddress` is updated whenever `Address` changes while `CopyAddress"` remains `true`.

### Option 1: Use `setValueExpression` Instead

Replace the survey-level trigger with the [`setValueIf`](https://surveyjs.io/form-library/documentation/api-reference/question#setValueIf) and [`setValueExpression`](https://surveyjs.io/form-library/documentation/api-reference/question#setValueExpression) properties on `BillingAddress` for a simpler, more maintainable setup:

```json
{
  "elements": [
    { "type": "comment", "name": "Address", "title": "Address", "isRequired": true },
    { "type": "boolean", "name": "CopyAddress", "title": "Use the address for billing", "defaultValue": false },
    {
      "type": "comment",
      "name": "BillingAddress",
      "title": "Billing Address",
      "setValueIf": "{CopyAddress} = true",
      "setValueExpression": "{Address}",
      "enableIf": "{CopyAddress} = false"
    }
  ],
  "textUpdateMode": "onTyping"
}
```

**How it works:**

- `setValueIf` activates when `CopyAddress` is `true`.
- `setValueExpression` copies the value of `Address` to `BillingAddress`.
- [`enableIf`](https://surveyjs.io/form-library/documentation/api-reference/question#enableIf) disables the `BillingAddress` input when `CopyAddress` is `true` for better UX.
- `defaultValue: false` on `CopyAddress` avoids undefined states, allowing `enableIf: "{CopyAddress} = false"`.

### Option 2: Include `Address` in the Trigger Expression

Make the trigger also depend on `Address` so that it re-evaluates when that value changes:

```json
"expression": "{CopyAddress} = true and {Address} notempty"
```

Full example:


```json
{
  "elements": [
    { "type": "comment", "name": "Address", "title": "Address", "isRequired": true },
    { "type": "boolean", "name": "CopyAddress", "title": "Use the address for billing" },
    { "type": "comment", "name": "BillingAddress", "title": "Billing Address" }
  ],
  "triggers": [
    {
      "type": "copyvalue",
      "fromName": "Address",
      "setToName": "BillingAddress",
      "expression": "{CopyAddress} = true and {Address} notempty"
    }
  ],
  "textUpdateMode": "onTyping"
}
```

If `Address` can be empty, use:

```json
"expression": "{CopyAddress} = true and ({Address} notempty or {Address} empty)"
```

### Option 3: Extend `copyvalue` Trigger Behavior

Create a custom trigger class that also reacts when the `fromName` question changes:

```js
import { Serializer, SurveyTriggerCopyValue } from "survey-core";

class SurveyTriggerCopyValueEx extends SurveyTriggerCopyValue {
  protected getUsedVariables(): string[] {
    const res = super.getUsedVariables();
    res.push(this.fromName); // Include the source question in trigger evaluation
    return res;
  }
}

// Replace the default `copyvalue` trigger with the custom one
const copyClassInfo = Serializer.findClass("copyvaluetrigger");
copyClassInfo.creator = () => new SurveyTriggerCopyValueEx();
```

This approach preserves the trigger-based workflow but fixes the limitation by making `Address` part of the re-evaluation process automatically.

### When to Use Each Option

- **Option 1 (`setValueExpression`)**\
Best for new forms or when you control the survey JSON. It's concise, reactive to all changes, and requires no custom code.

- **Option 2 (expression update)**\
Easiest fix if you want to keep using triggers without changing question definitions. Works well for small adjustments.

- **Option 3 (custom trigger)**\
Ideal if you have multiple `copyvalue` triggers affected by the same limitation and want a global fix without rewriting each expression.

<!-- ## Related Tags
- surveyjs
- copyvalue
- setValueExpression
- survey-json
- javascript -->