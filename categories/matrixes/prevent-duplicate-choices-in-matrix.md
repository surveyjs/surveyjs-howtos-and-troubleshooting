# Prevent Duplicate Choices in Matrix Dropdown Columns

## Problem
How to hide or disable choices that are already selected in matrix dropdown columns to prevent users from selecting duplicate choices?

## Solution
There are several approaches to prevent duplicate choice selection in matrix dropdown columns.

**Option 1: Validation with `isUnique`**  
Enable the [`isUnique`](https://surveyjs.io/form-library/documentation/api-reference/multi-select-matrix-column-values#isUnique) option for the matrix column to validate uniqueness:
```js
{
  "name": "Phone Model",
  "cellType": "dropdown", 
  "isUnique": true
}
```

**Option 2: Using onPopupVisibleChanged Event**  
Subscribe to the [`SurveyModel.onPopupVisibleChanged`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onPopupVisibleChanged) event and modify the choices collection to disable selected choices:
```js
const selectedInMatrix = (matrixValue, choiceValue) => { 
  for (const key of Object.keys(matrixValue)) { 
    const value = matrixValue[key]; 
    if (value['Column 1'] === choiceValue) { 
      return true; 
    } 
  } 
  return false; 
};

survey.onPopupVisibleChanged.add((sender, options) => { 
  if (!options.visible) return; 
  const matrixValue = survey.getValue('question1'); 
  if (!!matrixValue) { 
    for (let i = 0; i < options.question.choices.length; i++) { 
      let choice = options.question.choices[i]; 
      const isAlreadySelected = selectedInMatrix(matrixValue, choice.value); 
      choice.enabled = !isAlreadySelected; 
    } 
  } 
});
```

**Option 3: Lazy Loading with `enableIf`**  
Enable [`QuestionDropdownModel.choicesLazyLoadEnabled`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#choicesLazyLoadEnabled) and programmatically load choices using [`SurveyModel.onChoicesLazyLoad`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onChoicesLazyLoad). To disable a choice, check if a choice is already selected and disable its `enableIf` option.
```js
const json = {
  "type": "matrixdropdown",
  "name": "question1",
  "columns": [{
    "name": "Column 1",
    "cellType": "dropdown",
    "choicesLazyLoadEnabled": true
  }],
  "rows": ["Row 1", "Row 2", "Row 3"]
};

survey.onChoicesLazyLoad.add((sender, options) => {
  const matrixValue = sender.getValue('question1');
  const newChoices = choiceItems.slice();
  if (!!matrixValue) {
    for (let i = 0; i < newChoices.length; i++) {
      let choice = newChoices[i];
      const isAlreadySelected = selectedInMatrix(matrixValue, choice.value);
      choice.enableIf = !isAlreadySelected;
    }   
  }
  options.setItems(newChoices);
});
```
