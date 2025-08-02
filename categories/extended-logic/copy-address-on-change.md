# Copy Address Value When Checkbox Is Already Set in SurveyJS

## Question
In a SurveyJS form, we use a `copyvalue` trigger to copy the "address" question value into "billing_address" when the "copy_address" boolean question is set to `true`. This works when toggling "copy_address," but if "copy_address" is already checked and the "address" value changes, the trigger doesn’t update "billing_address." How can I ensure the "billing_address" updates when "address" changes while "copy_address" is checked?

### Original JSON
```json
{
  "elements": [
    { "type": "comment", "name": "address", "title": "Address", "isRequired": true },
    { "type": "boolean", "name": "copy_address", "title": "Use the address for billing" },
    { "type": "comment", "name": "billing_address", "title": "Billing Address" }
  ],
  "triggers": [
    { "type": "copyvalue", "fromName": "address", "setToName": "billing_address", "expression": "{copy_address} = true" }
  ],
  "textUpdateMode": "onTyping"
}
```

## Answer
The `copyvalue` trigger in SurveyJS copies the value from `fromName` to `setToName` when the `expression` evaluates to `true`. However, the trigger only runs when variables in the `expression` (e.g., `copy_address`) change, not when the `fromName` question (e.g., `address`) changes. Below are three solutions to ensure "billing_address" updates when "address" changes while "copy_address" is `true`.

### Solution 1: Include "address" in Trigger Expression
Modify the `copyvalue` trigger to include the `address` question in the `expression`. This ensures the trigger re-evaluates when `address` changes.

```json
{
  "triggers": [
    {
      "type": "copyvalue",
      "fromName": "address",
      "setToName": "billing_address",
      "expression": "{copy_address} = true and {address} notempty"
    }
  ]
}
```

Since `address` is required (`isRequired: true`), the `notempty` condition is sufficient. To handle cases where `address` might be empty, use:

```json
{
  "triggers": [
    {
      "type": "copyvalue",
      "fromName": "address",
      "setToName": "billing_address",
      "expression": "{copy_address} = true and ({address} notempty or {address} empty)"
    }
  ]
}
```

**Pros**: Simple, no code changes needed.  
**Cons**: Less flexible for complex logic.

### Solution 2: Extend Copy Value Trigger Behavior
Customize the `SurveyTriggerCopyValue` class to make the trigger re-evaluate when the `fromName` question (e.g., `address`) changes. Register a custom serializer in your application.

```typescript
import { Serializer, SurveyTriggerCopyValue } from "survey-core";

class SurveyTriggerCopyValueEx extends SurveyTriggerCopyValue {
  protected getUsedVariables(): string[] {
    const res = super.getUsedVariables();
    res.push(this.fromName); // Include fromName in trigger evaluation
    return res;
  }
}

// Replace default copyvalue trigger creator
const copyClassInfo = Serializer.findClass("copyvaluetrigger");
copyClassInfo.creator = () => new SurveyTriggerCopyValueEx();
```

**Pros**: Reusable across all `copyvalue` triggers in your app.  
**Cons**: Requires custom code, which may not suit non-developers.

### Solution 3: Use setValueExpression (Recommended)
Instead of a survey-level `copyvalue` trigger, use the question-level `setValueIf` and `setValueExpression` properties on `billing_address`. This approach is simpler, more maintainable, and recommended by SurveyJS.

```json
{
  "elements": [
    { "type": "comment", "name": "address", "title": "Address", "isRequired": true },
    { "type": "boolean", "name": "copy_address", "title": "Use the address for billing", "defaultValue": false },
    {
      "type": "comment",
      "name": "billing_address",
      "title": "Billing Address",
      "setValueIf": "{copy_address} = true",
      "setValueExpression": "{address}",
      "enableIf": "{copy_address} = false"
    }
  ],
  "textUpdateMode": "onTyping"
}
```

- `setValueIf`: Activates when `copy_address` is `true`.
- `setValueExpression`: Copies the value of `address` to `billing_address`.
- `enableIf`: Disables the `billing_address` input when `copy_address` is `true` for better UX.
- `defaultValue: false` on `copy_address` avoids undefined states, allowing `enableIf: "{copy_address} = false"`.

**Pros**: Clean, question-level solution; improves UX with `enableIf`; no custom code needed.  
**Cons**: Requires SurveyJS version supporting `setValueExpression`.

## Recommended Approach
Use **Solution 3** (`setValueExpression`) for its simplicity, maintainability, and better user experience. It avoids survey-level triggers and is easier to manage in SurveyJS Creator.

## Related Tags
- surveyjs
- copyvalue
- setValueExpression
- survey-json
- javascript