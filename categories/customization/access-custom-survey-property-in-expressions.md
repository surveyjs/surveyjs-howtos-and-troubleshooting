# Access Custom Survey-Level Property in Expressions for Conditional Logic in SurveyJS

## Question
How can I access a custom survey-level property in expressions within SurveyJS (e.g., for conditional visibility like `visibleIf`), when the property is added using code and set at runtime?

### Example Custom Property Code
```typescript
import { Serializer } from "survey-core";

Serializer.addProperty("survey", {
  type: "dropdown",
  name: "transactionType",
  category: "general",
  choices: ["REQ", "END", "REN"],
  default: "REQ",
  visibleIndex: 3
});
```

## Answer
Direct access to custom survey-level properties (added via `Serializer.addProperty("survey", ...)`) in expressions like `visibleIf`, `enableIf`, or `requiredIf` is not supported by built-in functions such as `propertyValue`, which is limited to question-level properties (e.g., `propertyValue('question1', 'visible')`). Survey-level properties are part of the survey model but not directly available as placeholders like `{propertyName}` in expressions.

To access them, implement a custom function that retrieves the property value using `this.survey` (the survey instance available within custom function contexts). Register the function with `FunctionFactory`, then use it in expressions.

### Implementation: Custom Function to Access Survey Properties
Create and register a custom function like `getSurveyProperty` that accepts the property name as a parameter and returns its value.

```typescript
import { FunctionFactory } from "survey-core";

// Custom function to access any survey-level property
function getSurveyProperty(params: any[]): any {
  if (params.length < 1) return undefined;
  const propName = params[0];
  return this.survey[propName]; // Access via this.survey (survey model instance)
}

// Register the function for use in expressions
FunctionFactory.Instance.register("getSurveyProperty", getSurveyProperty);
```

### Usage in Survey JSON (e.g., for Visibility)
Once registered, use the function in expressions. For example, show a question only if `transactionType` is `'REQ'`:

```json
{
  "elements": [
    {
      "type": "text",
      "name": "specificQuestion",
      "title": "Specific Question (Visible if transactionType = REQ)",
      "visibleIf": "getSurveyProperty('transactionType') = 'REQ'"
    }
  ]
}
```

- Set the property at runtime (e.g., after survey creation):
  ```typescript
  const survey = new Model(surveyJson);
  survey.transactionType = "REQ"; // Set custom property value
  ```

### Alternative: Use Variables Instead of Custom Properties
If possible, use survey variables (`survey.setVariable('name', value)`) instead of custom properties. Variables are directly accessible in expressions as `{variableName}` without custom functions.

```typescript
const survey = new Model(surveyJson);
survey.setVariable("transactionType", "REQ");
```

In JSON:
```json
{
  "elements": [
    {
      "type": "text",
      "name": "specificQuestion",
      "visibleIf": "{transactionType} = 'REQ'"
    }
  ]
}
```

**Pros of Variables**: Simpler direct access in expressions; automatically included if `includeIntoResult` is set for calculated values.  
**Cons**: Not editable in Property Grid like custom properties; less suitable for design-time configuration.

### Additional Option: Access in Custom Functions for Question Properties
If the expression is question-specific, use `this.question` alongside `this.survey` in custom functions (e.g., to combine survey and question properties).

**Limitations**:
- Custom functions run at runtime; ensure registration before survey creation.
- `propertyValue` does not support survey-level; it's for questions only.
- For async needs, make the function asynchronous with `this.returnResult()`.

## Recommended Approach
Use a custom function like `getSurveyProperty` for flexibility with survey-level properties. If the property doesn't need Property Grid editing, switch to variables for simpler expression access.

## Related Tags
- surveyjs
- custom-property
- expressions
- visibleIf
- custom-function
- javascript

## References
- [SurveyJS Conditional Logic Documentation](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic) – Covers expressions, custom functions, and `propertyValue`.
- [SurveyJS Custom Functions](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#custom-functions)  – Implementing and registering functions with `this.survey`.
- [SurveyJS Add Custom Properties](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form) – Adding properties to survey or questions.