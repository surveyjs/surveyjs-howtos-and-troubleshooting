# How to apply a custom currency mask to matrix dropdown column totals

## Problem

When you configure a column in a **matrixdropdown** question with:

- `totalType: "sum"`
- `totalDisplayStyle: "currency"`
- `maskType: "currency"` + custom `maskSettings` (e.g. decimal separator `,`, thousands separator ` `, suffix ` €`)

the total value in the footer row may not display with your exact custom formatting (especially the separators and suffix). The built-in currency formatting does not always pick up the exact mask configuration defined on the column.

## Solution

Subscribe to the `onGetExpressionDisplayValue` event on the Survey Model.  
When the question's `displayStyle` is `"currency"`, manually format the value using `InputMaskCurrency` from `survey-core`.

This approach works for matrix column totals (and other expression-based display values).

## Example

**JavaScript / TypeScript (React example shown)**

```tsx
import { Survey } from "survey-react-ui";
import { Model } from "survey-core";
import { InputMaskCurrency } from "survey-core";   // or "survey-react" depending on your setup

function SurveyComponent() {
  const survey = new Model();

  // Create and configure the currency mask
  const currencyMask = new InputMaskCurrency();
  currencyMask.decimalSeparator = ",";
  currencyMask.thousandsSeparator = " ";
  currencyMask.suffix = " €";

  survey.onGetExpressionDisplayValue.add((sender, options) => {
    const q = options.question;
    // Apply custom mask only for questions/columns with displayStyle === "currency"
    if (q && q.displayStyle === "currency") {
      options.displayValue = currencyMask.getMaskedValue(options.value);
    }
  });

  // Load survey JSON *after* registering the event handler
  survey.fromJSON(json);

  survey.onComplete.add((sender) => {
    console.log(JSON.stringify(sender.data, null, 3));
  });

  return <Survey model={survey} />;
}
```

**Generic survey JSON** (no customer-specific data)

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
          "totalText": "Total"
        }
      ]
    }
  ]
}
```

## Notes

- The event handler must be registered **before** calling `survey.fromJSON(...)`.
- `InputMaskCurrency` is available in the `survey-core` package.
- You can extend this logic to other display styles or specific question names if needed.
- This solution ensures the total row displays exactly as defined by your mask (e.g., `1 234,56 €`).

Feel free to adjust separators, suffix, or add prefix as required.
```

This structure follows the style of other how-tos in the repository: clear **Problem** → **Solution** → **Example** (with generic JSON) → **Notes**.

You can now create a new `.md` file under `categories/expressions/` (or the closest matching folder) and open a pull request with this content.