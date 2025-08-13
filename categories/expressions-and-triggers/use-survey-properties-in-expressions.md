# Use Survey-Level Properties in Expressions for Conditional Logic

## Problem

I have a custom survey-level property and want to use it (along with other survey-level properties) in expressions like `visibleIf` to configure conditional logic.

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

## Solution

In the SurveyJS Form Library, expressions can directly access **question-level** properties using the `propertyValue` function (e.g., `propertyValue('question1', 'visible')`). To use a **survey-level** property in an expression, you need to create and register a custom function as follows:

1. Implement a custom function that retrieves the property value. Use `this.survey` and `this.question` to access the survey and question instances.
2. Register the function in `FunctionFactory`.
3. Use the function in expressions such as `visibleIf`.

### Code Sample: Create and Register the Function

```typescript
import { FunctionFactory } from "survey-core";

// Step 1: Implement a custom function that retrieves the property value
function getSurveyProperty(params: any[]): any {
  if (params.length < 1) return undefined;
  const propName = params[0];
  return this.survey[propName];
  // If async, call `this.returnResult(result)` instead of returning
}

// Step 2: Register the function in `FunctionFactory`
FunctionFactory.Instance.register("getSurveyProperty", getSurveyProperty);
```

### Survey JSON Schema

```json
{
  "elements": [
    {
      "type": "text",
      "name": "specificQuestion",
      "title": "Specific Question (Visible if transactionType = REQ)",
      // Step 3: Use the function in an expression
      "visibleIf": "getSurveyProperty('transactionType') = 'REQ'"
    }
  ]
}
```

### Code Sample: Set the Property at Runtime

```typescript
import { Model } from "survey-core";
const surveyJson = { /* Survey JSON schema */}
const survey = new Model(surveyJson);

// Set the custom survey-level property at runtime
survey.transactionType = "REQ";
```

## Alternative Solution

SurveyJS also supports **custom variables**, which can be used in expressions directly as `{variableName}` without writing a custom function. Use this approach if you don't need the property to appear in the Survey Creator's Property Grid.

### Code Sample 

```typescript
import { Model } from "survey-core";

const surveyJson = {
  "elements": [
    {
      "type": "text",
      "name": "specificQuestion",
      "visibleIf": "{transactionType} = 'REQ'"
    }
  ]
};

const survey = new Model(surveyJson);
survey.setVariable("transactionType", "REQ");
```

## Learn More

- [Conditional Logic](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic)
- [Custom Functions](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#custom-functions)
- [Add Custom Properties](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)

<!-- ## Related Tags
- surveyjs
- custom-property
- expressions
- visibleIf
- custom-function
- javascript -->