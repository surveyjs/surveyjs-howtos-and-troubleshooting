# Create a Custom Question Adorner Action in SurveyJS Creator for Batch Editor

## Problem
In SurveyJS Creator, how to create a custom adorner action for a question to allow editors to activate the Batch Editor directly from the design surface?

## Solution
To add a custom adorner action that activates the Batch Editor for questions in SurveyJS Creator, implement the [`creator.onElementGetActions`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementGetActions) function. Within this function, register a custom action for instances of `QuestionSelectBase`. The action's click event should retrieve the property grid's "Edit" action and programmatically invoke its `action` function.

### Code Sample
```javascript
import { ComputedUpdater, QuestionSelectBase, Action } from "survey-core";

creator.onElementGetActions.add((_, options) => {
  const question = options.element;
  if (question instanceof QuestionSelectBase) {
    const bulkEditAction = new Action({
      id: "bulkEdit",
      active: new ComputedUpdater(() => question.readOnly),
      title: "Bulk Edit",
      iconName: "icon-textedit-24x24",
      action: () => {
        const propQuestion = creator.propertyGrid.getQuestionByName("choices");
        const action = propQuestion
          .getTitleToolbar()
          .getActionById("property-grid-setup");
        if (action) action.action();
      },
      visibleIndex: 9,
    });
    options.actions.push(bulkEditAction);
  }
});
```