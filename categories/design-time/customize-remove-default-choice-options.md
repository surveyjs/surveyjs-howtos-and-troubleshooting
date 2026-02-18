# Remove or Customize Default Choices in Choice-Based Questions

## Problem
By default, Survey Creator toolbox items like dropdowns, checkboxes, radiogroups, and ranking questions come with predefined choices like `"Item 1"`, `"Item 2"`, `"Item 3"`, special choices such as "Show "None" Item", "Show "Other" Item", and "Show "Select All" Item". This can be inconvenient if you want your users to start with empty options or with your own custom choices.  

How to remove or customize default choices for dropdown questions for all choice-based questions which extend [`QuestionSelectBase`](https://surveyjs.io/form-library/documentation/api-reference/questionselectbase), such as [Radiogroup](https://surveyjs.io/form-library/documentation/api-reference/radio-button-question-model), [Dropdown](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model), and [Checkbox](https://surveyjs.io/form-library/documentation/api-reference/checkbox-question-model)?

Additionally, developers may want to:
- Remove the **None** and **Other** items.
- Hide the **Select All** option.
- Hide the new item option.
- Customize newly added questions dynamically.

## Solution
You can control default behavior for all choice-based questions in several ways:

1. Remove or customize default choices in the toolbox configuration.
2. Hide **None**, **Other**, and **Select All** options using the Serializer API.
3. Customize questions when they are added using the [`creator.onQuestionAdded`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onQuestionAdded) event.
4. Hide specific items at runtime using survey model events.

### Code Sample

#### 1. Remove default choices from all toolbox items

```javascript
import { settings } from "survey-creator-core";

const defaultJSON = settings.toolbox.defaultJSON;

for (const key in defaultJSON) {
  if (defaultJSON[key] && Array.isArray(defaultJSON[key].choices)) {
    defaultJSON[key].choices = [];
  }
}
```

#### 2. Customize default choices

```javascript
import { settings } from "survey-creator-core";

const defaultJSON = settings.toolbox.defaultJSON;

defaultJSON.dropdown.choices = ["Option A", "Option B"];
defaultJSON.checkbox.choices = ["Yes", "No"];
defaultJSON.radiogroup.choices = ["Red", "Green", "Blue"];
defaultJSON.ranking.choices = ["High", "Medium", "Low"];
```

#### 3. Remove 'None' and 'Other' options from the design surface and property grid

```javascript
import { Serializer } from "survey-core";

Serializer.getProperty("selectbase", "showNoneItem").visible = false;
Serializer.getProperty("selectbase", "showOtherItem").visible = false;
```

#### 4. Hide 'Select All' for Checkbox and TagBox

```javascript
Serializer.getProperty("checkbox", "showSelectAllItem").visible = false;
Serializer.getProperty("tagbox", "showSelectAllItem").visible = false;
```

#### 5. Customize questions when they are added

```javascript
creator.onQuestionAdded.add((sender, options) => {
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

#### 6. Hide the new choice item at runtime

```javascript
creator.onSurveyInstanceSetupHandlers.add(function (sender, options) {
  if (options.area === "designer-tab") {
    options.survey.onShowingChoiceItem.add(function (sender, options) {
      if (options.item.value === "newitem") {
        options.visible = false;
      }
    });
  }
});
```