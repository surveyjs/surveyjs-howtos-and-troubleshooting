# Create a Custom Adorner Action in SurveyJS Creator

## Problem

In SurveyJS Creator, I want to add a custom adorner action that allows authors to open the batch choice editor directly from the design surface.

## Solution

To create a custom adorner action, handle the [`onElementGetActions`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementGetActions) event. Inside the event handler, define a new `Action` for elements that inherit from `QuestionSelectBase` (these are the question types that support choices). When the user clicks this action, it should programmatically trigger the Property Grid's built-in **Edit** action for the `choices` property, which opens the batch editor.

### Code Sample

```js
import { ComputedUpdater, QuestionSelectBase, Action } from "survey-core";

// ...
// Omitted: `SurveyCreatorModel` creation
// ...

creator.onElementGetActions.add((_, options) => {
  const question = options.element;
  if (question instanceof QuestionSelectBase) {
    const batchEditAction = new Action({
      id: "batchEdit",
      title: "Batch Edit",
      iconName: "icon-textedit-24x24",
      active: new ComputedUpdater(() => question.readOnly),
      action: () => {
        const propQuestion = creator.propertyGrid.getQuestionByName("choices");
        const editAction = propQuestion
          ?.getTitleToolbar()
          ?.getActionById("property-grid-setup");
        editAction?.action();
      },
      visibleIndex: 9,
    });
    options.actions.push(batchEditAction);
  }
});
```