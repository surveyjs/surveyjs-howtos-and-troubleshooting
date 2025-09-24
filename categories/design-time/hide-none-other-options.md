# Hide the 'Other' and 'None' options in Survey Creator

## Problem
When designing surveys in SurveyJS Creator, users should not be able to add "Other" or "None" options to choice-based questions such as [Radiogroup](https://surveyjs.io/form-library/documentation/api-reference/radio-button-question-model), [Checkboxes](https://surveyjs.io/form-library/documentation/api-reference/checkbox-question-model), [Dropdown](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model). These special options (`showOtherItem` and `showNoneItem`) need to be hidden from both the Property Grid and the Design Surface.

## Solution
Use SurveyJS Creator's event handlers to control visibility:
- **Property Grid**: Use the [`onPropertyShowing`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyShowing) event to hide the `showOtherItem` and `showNoneItem` properties.
- **Design Surface**: Use the [`onSurveyInstanceSetupHandlers`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onSurveyInstanceSetupHandlers) event to obtain a design-time survey instance. Subscribe to [`SurveyModel.onShowingChoiceItem`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onShowingChoiceItem) function and hide "Other" and "None" choice items for questions in the designer tab.

### Code Sample
```javascript
// Hide properties in Property Grid
const propsToHide = ["showOtherItem", "showNoneItem"];
creator.onPropertyShowing.add((sender, options) => {
  if (propsToHide.indexOf(options.property.name) >= 0) {
    options.show = false;
  }
});

// Hide Other and None items on Design Surface
creator.onSurveyInstanceSetupHandlers.add((sender, options) => {
  if (options.area === "designer-tab") {
    options.survey.onShowingChoiceItem.add((sender, options) => {
      if (["other", "none"].indexOf(options.item.value) >= 0 && options.question instanceof Survey.QuestionSelectBase) {
        options.visible = false;
      }
    });
  }
});
```