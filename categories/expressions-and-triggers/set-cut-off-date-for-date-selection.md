# Set a Cut-Off Time for Date Selection

## Problem

Sometimes in surveys or forms, you want to restrict users from selecting certain dates based on the current time.

For example, you might want to prevent users from picking tomorrow if it is already after 2 PM today. This ensures that responses are only collected for dates that are logically allowed.

## Solution

To implement this in SurveyJS, implement a [custom function](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#implement-a-custom-function) and use the [`minValueExpression`](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model#minValueExpression) property of a date/datetime question.

Steps:
* Check the current time.
* If it’s after the cutoff hour (2 PM), move the minimum selectable date to the day after tomorrow.
* Otherwise, allow tomorrow as the earliest selectable date.

### Code Sample

```javascript
import { FunctionFactory } from "survey-core";

// Register a custom function for cutoff date
FunctionFactory.Instance.register("cutoffMinDate", function () {
  const now = new Date();
  const minDate = new Date();

  if (now.getHours() >= 14) {
    // After 2 PM → earliest selectable date is day after tomorrow
    minDate.setDate(minDate.getDate() + 2);
  } else {
    // Before 2 PM → earliest selectable date is tomorrow
    minDate.setDate(minDate.getDate() + 1);
  }

  // Return in ISO format for datetime-local input
  return minDate.toISOString().slice(0, 16);
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
          "name": "question4",
          "title": "Select a date",
          "inputType": "datetime-local",
          "minValueExpression": "cutoffMinDate()"
        }
      ]
    }
  ]
}
```