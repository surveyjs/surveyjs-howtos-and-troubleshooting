# Limit a Custom Component to One Instance and Make Its Name Read-Only

## Problem
How to ensure that a custom component or question in a survey is added only once and prevent users from modifying its name?

## Solution
To limit a custom component to a single instance, use the [`creator.onQuestionAdded`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onQuestionAdded) event and call [`QuestionToolbox.removeItem`](https://surveyjs.io/survey-creator/documentation/api-reference/questiontoolbox#removeItem) remove the component from the toolbox after it is added, and set its name programmatically. 

If you load an existing survey, implement the [`creator.onSurveyInstanceCreated`)]((https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onSurveyInstanceCreated)) function and check whether a target element exists. Disable the toolbox item if it exists. To re-enable the component in the toolbox when it is removed, use the [`creator.onElementDeleting`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementDeleting) event. To make the component's name read-only, use the [`creator.onPropertyGetReadOnly`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyGetReadOnly) event to set the `name` property as read-only for the specific component type.

## Code Sample
```js
// Disable the toolbox item if the widget already exists when loading
creator.onSurveyInstanceCreated.add((sender, options) => {
  if (options.area === 'designer-tab') {
    const existing = options.survey
      .getAllQuestions(false, true, true)
      .find((q) => q.name === 'UniqueItem' && q.getType() === 'uniquewidget');
    if (existing) {
      const toolboxItem = sender.toolbox.getItemByName('uniquewidget');
      toolboxItem.enabled = false;
    }
  }
});

// When added, set the name and disable further addition
creator.onQuestionAdded.add((sender, options) => {
  if (options.question.getType() === 'uniquewidget') {
    options.question.name = 'UniqueItem';
    const toolboxItem = sender.toolbox.getItemByName('uniquewidget');
    toolboxItem.enabled = false;
  }
});

// Re-enable the widget in the toolbox if it is removed
creator.onElementDeleting.add((sender, options) => {
  if (options.elementType === 'question') {
    const removed = options.element;
    if (removed.getType() === 'uniquewidget') {
      const toolboxItem = sender.toolbox.getItemByName('uniquewidget');
      toolboxItem.enabled = true;
    }
  }
});

// Make the name property read-only
creator.onPropertyGetReadOnly.add((sender, options) => {
  if (options.property.name === "name" && options.element.getType() === "uniquewidget") {
    options.readOnly = true;
  }
});
```