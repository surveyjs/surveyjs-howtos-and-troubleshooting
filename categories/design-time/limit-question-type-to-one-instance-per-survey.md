# Limit a Question Type to One Instance per Survey and Make Its Name Read-Only

## Problem

I want to allow only one question of a specific type in a survey and prevent users from renaming it.

## Solution

Use Survey Creator API events to enforce these restrictions:

1. **Disable the toolbox item when the survey loads**\
In the [`onSurveyInstanceCreated`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onSurveyInstanceCreated) event handler, check if the question type already exists. If it does, disable its toolbox item.
2. **Manage toolbox item availability when questions change**\
Use the [`onQuestionAdded`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onQuestionAdded) and [`onElementDeleting`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementDeleting) events to disable the toolbox item when the question is added and re-enable it when the question is removed.
3. **Make the `name` property read-only**\
In the [`onPropertyGetReadOnly`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyGetReadOnly) event handler, detect the `name` property and enable the `readOnly` flag for questions of the required type.

## Code Sample

```js
// Step 1: Disable the toolbox item when the survey loads
creator.onSurveyInstanceCreated.add((_, options) => {
  if (options.area === 'designer-tab') {
    const qExists = options.survey
      .getAllQuestions(false, true, true)
      .find((q) => q.name === 'UniqueQuestion' && q.getType() === 'uniquequestion');
    if (qExists) {
      const toolboxItem = creator.toolbox.getItemByName('uniquequestion');
      toolboxItem.enabled = false;
    }
  }
});

// Step 2: Manage toolbox item availability when questions change
creator.onQuestionAdded.add((_, options) => {
  if (options.question.getType() === 'uniquequestion') {
    options.question.name = 'UniqueQuestion';
    const toolboxItem = creator.toolbox.getItemByName('uniquequestion');
    toolboxItem.enabled = false;
  }
});
creator.onElementDeleting.add((_, options) => {
  if (options.elementType === 'question') {
    const removed = options.element;
    if (removed.getType() === 'uniquequestion') {
      const toolboxItem = creator.toolbox.getItemByName('uniquequestion');
      toolboxItem.enabled = true;
    }
  }
});

// Step 3: Make the `name` property read-only
creator.onPropertyGetReadOnly.add((_, options) => {
  if (options.property.name === "name" && options.element.getType() === "uniquequestion") {
    options.readOnly = true;
  }
});
```

## Learn More

- [Documentation: Customize Survey Elements on Creation](https://surveyjs.io/survey-creator/documentation/customize-survey-creation-process#customize-survey-elements-on-creation)
- [Demo: Modify New Question](https://surveyjs.io/survey-creator/examples/dynamically-modify-newly-added-questions/)