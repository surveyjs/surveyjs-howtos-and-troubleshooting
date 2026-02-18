# Remove or Customize Default Choices in Choice-Based Questions

## Problem

In Survey Creator, choice-based questions such as [Dropdown](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model), [Checkboxes](https://surveyjs.io/form-library/documentation/api-reference/checkbox-question-model), and [Radio Button Group](https://surveyjs.io/form-library/documentation/api-reference/radio-button-question-model) are initialized with predefined choice items. Authors can also enable special items&mdash;"None", "Other", and "Select All"&mdash;directly in the design surface. I want to remove the predefined choice options and disallow users to add the special choices.

## Solution

You can control the choice options in several ways:

- [Remove](#remove-default-choices) or [customize default choices](#customize-default-choices) in the toolbox configuration.
- [Hide the "None", "Other"](#hide-the-none-and-other-options), and ["Select All"](#hide-the-select-all-option-for-checkbox-and-tag-box) options using the `Serializer` API.
- [Customize questions on creation](#customize-questions-on-creation) using the Survey Creator's [`onQuestionAdded`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onQuestionAdded) event.
- [Hide specific items at runtime](#hide-the-new-choice-item-on-the-design-surface) using survey model events.

### Remove Default Choices

```javascript
import { settings } from "survey-creator-core";

const defaultJSON = settings.toolbox.defaultJSON;

for (const key in defaultJSON) {
  if (defaultJSON[key] && Array.isArray(defaultJSON[key].choices)) {
    defaultJSON[key].choices = [];
  }
}
```

### Customize Default Choices

```javascript
import { settings } from "survey-creator-core";

const defaultJSON = settings.toolbox.defaultJSON;

defaultJSON.dropdown.choices = ["Option A", "Option B"];
defaultJSON.checkbox.choices = ["Red", "Green", "Blue"];
defaultJSON.radiogroup.choices = ["Yes", "No"];
defaultJSON.ranking.choices = ["High", "Medium", "Low"];
```

### Hide the "None" and "Other" Options

```javascript
import { Serializer } from "survey-core";

Serializer.getProperty("selectbase", "showNoneItem").visible = false;
Serializer.getProperty("selectbase", "showOtherItem").visible = false;
```

### Hide the "Select All" Option (Checkbox and Tag Box)

```javascript
Serializer.getProperty("checkbox", "showSelectAllItem").visible = false;
Serializer.getProperty("tagbox", "showSelectAllItem").visible = false;
```

### Customize Questions on Creation

```javascript
// ...
// Omitted: `SurveyCreatorModel` creation
// ...
creator.onQuestionAdded.add((_, options) => {
  const question = options.question;

  if (question.getType && question.getType() === "dropdown") {
    question.choices = ["Custom 1", "Custom 2"];
  }

  if (question.showNoneItem !== undefined) {
    question.showNoneItem = false;
  }

  if (question.showOtherItem !== undefined) {
    question.showOtherItem = false;
  }
});
```

### Hide the "New Item" Placeholder on the Design Surface

```javascript
// ...
// Omitted: `SurveyCreatorModel` creation
// ...
creator.onSurveyInstanceSetupHandlers.add((_, options) => {
  if (options.area === "designer-tab") {
    options.survey.onShowingChoiceItem.add((_, options2) => {
      if (options2.item.value === "newitem") {
        options2.visible = false;
      }
    });
  }
});
```