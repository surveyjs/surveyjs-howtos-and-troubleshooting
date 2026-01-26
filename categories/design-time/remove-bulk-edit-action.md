# How to Disable Bulk Edit for Dropdown Choices in SurveyJS Creator

## Problem
In SurveyJS Creator, the **Dropdown** question type allows you to edit its choices. By default, the **Choices** property includes a **bulk edit** action, letting users modify multiple choices at once. In some cases, you may want to hide or remove this bulk edit functionality to restrict editing to individual items only.

## Solution
You can achieve this by using the [`creator.onConfigureTablePropertyEditor`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onConfigureTablePropertyEditor) event. This event allows you to configure property editors for table-like properties (such as choices). By setting `options.allowBatchEdit` to `false`, the bulk edit action is disabled.

### Code Sample
```javascript
// Disable bulk edit for the Choices property of dropdowns
creator.onConfigureTablePropertyEditor.add((sender, options) => {
  if (options.propertyName === "choices") {
    options.allowBatchEdit = false;
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
          "type": "dropdown",
          "name": "favoriteColor",
          "title": "Select your favorite color",
          "choices": ["Red", "Green", "Blue"]
        }
      ]
    }
  ]
}
```