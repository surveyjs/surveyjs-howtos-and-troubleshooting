# How to Show/Hide the Details Action for Each Choice in SurveyJS Creator

## Problem
In SurveyJS Creator, the **Choices** property of a question provides a **details action** for each item. This allows users to open a detailed editor for individual choices. In some scenarios, you may want to hide or remove this action to simplify the UI and prevent detailed editing.

## Solution
To remove the show/hide details item action, implement the [`creator.onSurveyInstanceSetupHandlers`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onSurveyInstanceSetupHandlers) function and subscribe to the [`SurveyModel.onGetMatrixRowActions`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onGetMatrixRowActions) function of a property grid survey.

### Code Sample
```javascript
// Remove details action for each choice in the property grid
creator.onSurveyInstanceSetupHandlers.add((sender, options) => {
  if (options.area === "property-grid") {
    options.survey.onGetMatrixRowActions.add((sender, options) => {
      if (options.question.name === "choices") {
        options.actions = [];
      }
    });
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