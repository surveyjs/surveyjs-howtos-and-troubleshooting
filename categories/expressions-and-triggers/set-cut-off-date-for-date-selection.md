# Set a Cut-Off Time for Date Selection

## Problem

I need to restrict available dates based on the current time. For example, if it is already 2:00 PM or later, users should not be able to select tomorrow. In this case, the earliest available date must automatically shift to the day after tomorrow.

## Solution

In SurveyJS, you can implement time-dependent date constraints by creating a [custom function](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#implement-a-custom-function) and assigning it to the question's [`minValueExpression`](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model#minValueExpression) property. The function should calculate the minimum selectable date based on the current time and update the question accordingly.

### Code Sample

```javascript
import { registerFunction } from "survey-core";

registerFunction({
  name: "cutoffMinDate",
  func: () => {
    const now = new Date();
    const minDate = new Date();

    if (now.getHours() >= 14) {
      // 2:00 PM or later → earliest selectable date is day after tomorrow
      minDate.setDate(minDate.getDate() + 2);
    } else {
      // Before 2:00 PM → earliest selectable date is tomorrow
      minDate.setDate(minDate.getDate() + 1);
    }

    // Return value in ISO format for datetime-local input...
    return minDate.toISOString().slice(0, 16);
    // ... or for date input
    // return minDate.toISOString().slice(0, 10).
  }
});
```

### Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "text",
          "name": "question1",
          "title": "Select a date",
          "inputType": "datetime-local",
          "minValueExpression": "cutoffMinDate()"
        }
      ]
    }
  ]
}
```

With this configuration, the minimum selectable date automatically adjusts depending on whether the current time is before or after the defined cut-off (2:00 PM in this example).