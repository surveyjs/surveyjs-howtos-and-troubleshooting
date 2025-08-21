# Clear Matrix Row Value When Disabled

## Problem

When a matrix row becomes disabled via its `enableIf` expression, SurveyJS does not automatically clear any answer already stored for that row. As a result, the disabled row's value may still be submitted with the survey. I need to ensure that row values are removed once the row is disabled.

## Solution

Use the `SurveyModel`'s `onPropertyValueChangedCallback`. This callback is triggered whenever a property of any survey element (including matrix rows) changes. By monitoring changes to a row's `isEnabled` property, I can detect when it becomes `false` and remove the corresponding value from the matrix data object.

In the survey JSON schema below, the `q2` matrix rows are conditionally enabled based on answers in `q1`. If a user selects **NO** in a `q1` row, the related row in `q2` is disabled, and its stored value is automatically cleared using `onPropertyValueChangedCallback`.

### Survey JSON Schema

```json
{
  "elements": [
    {
      "type": "matrix",
      "name": "q1",
      "title": "1. Does the Government have any law(s) or regulation(s) that guarantee the following services/rights?",
      "columns": [
        { "value": "yes", "text": "YES" },
        { "value": "no", "text": "NO" }
      ],
      "rows": [
        {
          "value": "a_hiv_counselling",
          "text": "a. Voluntary HIV counselling and testing services"
        },
        {
          "value": "b_hiv_treatment",
          "text": "b. HIV treatment and care services"
        },
        {
          "value": "c_confidentiality",
          "text": "c. Protection of the confidentiality of all people living with HIV"
        }
      ]
    },
    {
      "type": "matrix",
      "name": "q2",
      "title": "2. If YES to 1a, 1b or 1c, are there any plural legal systems contradicting the above?",
      "columns": [
        { "value": "yes", "text": "YES" },
        { "value": "no", "text": "NO" }
      ],
      "rows": [
        {
          "value": "a_hiv_counselling",
          "text": "a. Voluntary HIV counselling and testing services",
          "enableIf": "{q1.a_hiv_counselling} = 'yes'"
        },
        {
          "value": "b_hiv_treatment",
          "text": "b. HIV treatment and care services",
          "enableIf": "{q1.b_hiv_treatment} = 'yes'"
        },
        {
          "value": "c_confidentiality",
          "text": "c. Protection of the confidentiality of all people living with HIV",
          "enableIf": "{q1.c_confidentiality} = 'yes'"
        }
      ]
    }
  ]
}
```

### Code Sample

```javascript
// ...
// Omitted: `Survey.Model` creation
// ...
survey.onPropertyValueChangedCallback = (name, _, newValue, sender) => {
  // When a row is disabled, clear its stored value
  if (name === "isEnabled" && newValue === false) {
    const matrix = sender.locOwner;
    if (!!matrix && matrix.getType() === "matrix" && !matrix.isEmpty()) {
      const val = matrix.value;
      delete val[sender.value]; // `sender.value` is the row name
      matrix.value = val; // Reassign to trigger change detection
    }
  }
};
```

[View Live Example](https://plnkr.co/edit/B2NBtWP8AcS0g01f)