# Multi-Select Matrix: Apply Currency Mask to Column Totals

## Problem

In a Multi-Select Matrix, a column is configured with a currency input mask (for example, decimal separator `,`, thousands separator ` `, and suffix ` €`).

While individual cell values are formatted correctly, the **column total** does not inherit these mask settings and is displayed as an unformatted number.

## Solution

Handle the [`onGetExpressionDisplayValue`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onGetExpressionDisplayValue) event on the [`SurveyModel`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model).

Column totals are calculated using expressions, so their display values bypass the input mask. To ensure consistent formatting, you need to **manually apply the same currency mask** when rendering expression-based values.

**Implementation steps:**

1. Create an `InputMaskCurrency` instance with the desired formatting rules.
2. Subscribe to `onGetExpressionDisplayValue`.
3. Detect values that should be rendered as currency (`displayStyle === "currency"`).
4. Replace the default display value with a masked value.

This approach ensures consistent formatting across matrix column totals and any other expression-based values.

## Code Sample

```tsx
import { Survey } from "survey-react-ui";
import { Model } from "survey-core";
import { InputMaskCurrency } from "survey-core";

function SurveyComponent() {
  const survey = new Model();

  // Configure a reusable currency mask
  const currencyMask = new InputMaskCurrency();
  currencyMask.decimalSeparator = ",";
  currencyMask.thousandsSeparator = " ";
  currencyMask.suffix = " €";

  // Apply mask to expression-based display values (for example, column totals)
  survey.onGetExpressionDisplayValue.add((_, options) => {
    const question = options.question;

    // Apply only to elements configured with currency display style
    if (question && question.displayStyle === "currency") {
      options.displayValue = currencyMask.getMaskedValue(options.value);
    }
  });

  // Load survey JSON *after* registering the event handler
  survey.fromJSON(json);

  return <Survey model={survey} />;
}
```

## Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "matrixdropdown",
          "name": "matrix",
          "title": "Matrix with currency total",
          "columns": [
            {
              "name": "col1",
              "title": "Column 1"
            },
            {
              "name": "col2",
              "title": "Column 2"
            },
            {
              "name": "amount",
              "title": "Amount",
              "cellType": "text",
              "totalType": "sum",
              "totalDisplayStyle": "currency",
              "maskType": "currency",
              "maskSettings": {
                "allowNegativeValues": false,
                "decimalSeparator": ",",
                "thousandsSeparator": " ",
                "min": 0,
                "max": 10000,
                "suffix": " €"
              }
            }
          ],
          "cellType": "text",
          "rows": [
            { "value": "row1", "text": "Row 1" },
            { "value": "row2", "text": "Row 2" }
          ],
          "totalText": "Total: "
        }
      ]
    }
  ]
}
```