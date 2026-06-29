# Validate How Many Times a Question Value Appears Across Dynamic Panels

## Problem

I want to validate that a specific question value appears no more than **N** times across all panels in a Dynamic Panel.

## Solution

If you only need to prevent duplicate values for a single question, use the [`keyName`](https://surveyjs.io/form-library/documentation/api-reference/dynamic-panel-model#keyName) property. This approach allows each value to appear only once.

If you need to allow a value to appear up to **N** times, add an Expression Validator to the Dynamic Panel and use the built-in [`countInArray`](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#countinarray) expression function.

`countInArray` counts items in a Dynamic Panel, [Dynamic Matrix](https://surveyjs.io/form-library/documentation/api-reference/dynamic-matrix-table-model), or [Multi-Select Matrix](https://surveyjs.io/form-library/documentation/api-reference/multi-select-matrix-table-model) array whose specified field satisfies an optional filter expression.

```js
countInArray(question: expression, valueField: string, filter?: expression)
```

- `question`: `expression`\
A reference to the Dynamic Panel question (for example, `{employees}`).

- `valueField`: `string`\
The name of the question within the panel template whose value should be evaluated.

- `filter`: `expression`\
*(Optional)* A Boolean expression evaluated for each panel or row. Only panels or rows for which the expression evaluates to `true` are counted.

To limit how many times a specific value can appear (for example, `"JavaScript"` selected in no more than three panels), use the `contains` operator in the filter expression.

> Attach the validator to the Dynamic Panel, not to a question inside its template. This ensures that validation is performed across all panels.

### Survey JSON Schema

The following example limits the `"JavaScript"` skill to no more than three employee records:

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "paneldynamic",
          "name": "employees",
          "title": "Employees",
          "validators": [
            {
              "type": "expression",
              "text": "You can select 'JavaScript' in no more than 3 employee records.",
              "expression": "countInArray({employees}, 'skills', {skills} contains 'JavaScript') <= 3"
            }
          ],
          "templateElements": [
            {
              "type": "checkbox",
              "name": "skills",
              "title": "Select skills",
              "choices": [
                "JavaScript",
                "TypeScript",
                "React",
                "Angular",
                "Vue"
              ]
            }
          ],
          "panelCount": 1,
          "displayMode": "tab"
        }
      ]
    }
  ]
}
```

## Learn More

- [Conditional Logic](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic)
- [Data Validation](https://surveyjs.io/form-library/documentation/data-validation)
- [Demo: Expressions in Dynamic Panel](https://surveyjs.io/form-library/examples/how-to-use-expressions-in-dynamic-panel/)