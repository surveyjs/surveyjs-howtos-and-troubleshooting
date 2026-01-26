# Remove Expandable Detail Sections for Choices in SurveyJS Creator

## Problem

In Survey Creator, each row in the Choices table includes an expandable detail section. I want to disable this feature so that the rows remain simple and unexpandable.

## Solution

Use the [`onSurveyInstanceSetupHandlers`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onSurveyInstanceSetupHandlers) event to access the survey instance that backs the Property Grid. Then, subscribe to the [`onGetMatrixRowActions`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onGetMatrixRowActions) event and remove the actions for the `choices` question.

### Code Sample

```js
// ...
// Omitted: `SurveyCreatorModel` creation
// ...
creator.onSurveyInstanceSetupHandlers.add((_, options1) => {
  if (options1.area === "property-grid") {
    options1.survey.onGetMatrixRowActions.add((_, options2) => {
      if (options2.question.name === "choices") {
        // Remove all expandable detail actions
        const newActions = options2.actions.filter(action => action.id !== "show-detail");
        options2.actions = newActions;
      }
    });
  }
});
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/nFccKeoBGtd6Mj3k)