# Prevent Duplicate Choice Selection in Matrix Dropdown Columns

## Problem

I want to prevent users from selecting the same choice more than once in matrix dropdown columns.

## Solution

There are several ways to block or discourage duplicate selection.

### Option 1: Validate Selected Choices with the `isUnique` Property

Enable the [`isUnique`](https://surveyjs.io/form-library/documentation/api-reference/multi-select-matrix-column-values#isUnique) option for a matrix column to enforce unique values. You can also set the `SurveyModel`'s [`checkErrorsMode`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#checkErrorsMode) property to `"onValueChanged"` if you want validation errors to appear immediately after a selection is made. 

```json
{
  "checkErrorsMode": "onValueChanged",
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "matrixdropdown",
          "name": "matrix-question-name",
          "columns": [
            {
              "name": "matrix-column-name",
              "cellType": "dropdown",
              "choices": [ /** An array of choice options */ ],
              "isUnique": true
            },
          ],
          "rows": [ /** An array of matrix rows */ ]
        }
      ]
    }
  ]
}
```

### Option 2: Disable Selected Choices with the `onPopupVisibleChanged` Event

Handle the `SurveyModel`'s [`onPopupVisibleChanged`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onPopupVisibleChanged) event to access the choice collection and dynamically disable choices that have already been selected elsewhere in the matrix column.

```json
{
  "pages": [
    {
      "name": "page-name",
      "elements": [
        {
          "type": "matrixdropdown",
          "name": "matrix-question-name",
          "columns": [
            {
              "name": "matrix-column-name",
              "cellType": "dropdown",              
              "choices": [ /** An array of choice options */ ]
            }
          ],
          "rows": [ /** An array of matrix rows */ ]
        }
      ]
    }
  ]
}
```

```js
const isChoiceSelectedInMatrix = (matrixData, choiceValue) => { 
  for (const key of Object.keys(matrixData)) {
    const value = matrixData[key];
    if (value['matrix-column-name'] === choiceValue) {
      return true;
    }
  }
  return false;
};

survey.onPopupVisibleChanged.add((_, options) => {
  if (!options.visible) return;
  const matrixData = survey.getValue('matrix-question-name');
  if (matrixData) {
    options.question.choices.forEach(choice => {
      choice.enabled = !isChoiceSelectedInMatrix(matrixData, choice.value);
    });
  }
});
```

### Option 3: Disable Choices During Lazy Loading

If you [lazy-load choices from a server](https://surveyjs.io/form-library/examples/lazy-loading-dropdown/), extend your [`onChoicesLazyLoad`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onChoicesLazyLoad) event handler to disable choices that have already been selected.

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "matrixdropdown",
          "name": "matrix-question-name",
          "columns": [
            {
              "name": "matrix-column-name",
              "cellType": "dropdown",
              "choicesLazyLoadEnabled": true
            }
          ],
          "rows": [ /** An array of matrix rows */ ]
        }
      ]
    }
  ]
}
```

```js
const isChoiceSelectedInMatrix = (matrixData, choiceValue) => { 
  for (const key of Object.keys(matrixData)) {
    const value = matrixData[key];
    if (value['matrix-column-name'] === choiceValue) {
      return true;
    }
  }
  return false;
};

survey.onChoicesLazyLoad.add((_, options) => {
  let choices = [];
  let totalCount = -1;
  // Fetch and set `choices` and `totalCount` from server here

  const matrixData = survey.getValue('matrix-question-name');
  const filteredChoices = choices.map(choice => ({
    ...choice,
    enabled: !matrixData || !isChoiceSelectedInMatrix(matrixData, choice.value)
  }));

  options.setItems(filteredChoices, totalCount);
});
```

## Learn More

- [Demo: Lazy Loading](https://surveyjs.io/form-library/examples/lazy-loading-dropdown/)