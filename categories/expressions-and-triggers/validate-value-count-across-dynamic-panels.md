# Validate How Many Times a Question Value Appears Across Dynamic Panels

## Problem

Is there a built-in way in SurveyJS to validate how many times a specific question value appears across all panels in a Dynamic Panel (or similar repeating structure)?

For example, given a Dynamic Panel with the same question repeated in each panel: can SurveyJS validate that a value appears no more than **N** times across all panels?

I know [`keyName`](https://surveyjs.io/form-library/documentation/api-reference/dynamic-panel-model#keyName) can be used to prevent duplicate responses for a question inside a Dynamic Panel, but is there a way to use expression-based validation to determine how many times a specific question value appears across panels?

## Solution

Yes. Use an **expression validator** on the Dynamic Panel together with the built-in [`countInArray`](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#countinarray) function.

`countInArray` counts items in a Dynamic Panel, Dynamic Matrix, or Multi-Select Matrix array where a specified field matches optional filter conditions. Attach a validator to the **panel dynamic question** (not to the inner template question) so the rule runs against all panels at once.

### How `countInArray` works

```text
countInArray({panelDynamicQuestion}, 'innerQuestionName', optionalFilterExpression)
```

- **First argument** — the Dynamic Panel question reference (for example, `{employees}`).
- **Second argument** — the name of a question inside the panel template whose value you want to evaluate.
- **Third argument** (optional) — a Boolean expression evaluated per panel/row. Only panels that satisfy the filter are counted.

To limit how often a specific choice appears (for example, `'JavaScript'` selected in no more than three panels), use the `allof` operator in the filter expression.

### Survey JSON Schema

The example below limits the `'JavaScript'` skill to at most three employee records:

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

Change the comparison (`<= 3`) and the filter expression to match your limit and value.


`countInArray` also works with [Dynamic Matrix](https://surveyjs.io/form-library/documentation/api-reference/dynamic-matrix-table-model) and [Multi-Select Matrix](https://surveyjs.io/form-library/documentation/api-reference/multi-select-matrix-table-model) when you need the same kind of cross-row counting.

### `keyName` vs expression validation

| Approach | Use when |
| --- | --- |
| [`keyName`](https://surveyjs.io/form-library/documentation/api-reference/dynamic-panel-model#keyName) on the Dynamic Panel | Each panel must have a **unique** value for one template question (duplicates are blocked entirely). |
| `countInArray` + expression validator | A value may repeat, but only up to **N** times (or you need a minimum count). |

## Learn More

- [Conditional Logic — Built-in Functions (`countInArray`)](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#countinarray)
- [Data Validation](https://surveyjs.io/form-library/documentation/data-validation)
- [Dynamic Panel Model API (`keyName`, `keyDuplicationError`)](https://surveyjs.io/form-library/documentation/api-reference/dynamic-panel-model)
- [Expressions in Dynamic Panel (example)](https://surveyjs.io/form-library/examples/how-to-use-expressions-in-dynamic-panel/documentation)
