# Limit Maximum Number of Selected Checkboxes in a Checkbox Matrix

## Problem

You are using a [checkbox matrix](https://surveyjs.io/form-library/examples/checkbox-matrix-question/reactjs) and want to set a maximum limit on the total number of checked cells across the entire matrix.

Examples of desired behavior:

- It's acceptable if a respondent checks **all checkboxes in one row** or **all checkboxes in one column**.
- It's **not acceptable** if the total number of checked cells in the whole matrix exceeds a certain number (e.g. 3, 5, 8…).

## Solution

Use the [`SurveyModel.onValidateQuestion`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onValidateQuestion) event to count all selected checkboxes in the matrix and set a custom error message when the total exceeds your limit.

### Code Sample

```javascript
// Set your desired maximum here
const MAX_TOTAL_SELECTIONS = 4;

survey.onValidateQuestion.add(function (sender, options) {
  // Change "question1" to your actual matrix question name
  if (options.name !== "question1") return;

  const value = options.value || {};
  let totalChecked = 0;

  // Loop through each row
  Object.keys(value).forEach(rowKey => {
    const selectedInRow = value[rowKey];
    
    // In checkbox matrix, value[row] is an array of selected column values
    if (Array.isArray(selectedInRow)) {
      totalChecked += selectedInRow.length;
    }
  });

  if (totalChecked > MAX_TOTAL_SELECTIONS) {
    options.error = `You can select maximum ${MAX_TOTAL_SELECTIONS} options in total across the whole matrix. You selected ${totalChecked}.`;
  }
});
```

## Survey JSON Schema
```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "matrix",
          "name": "question1",
          "title": "Please select the features that are important to you",
          "columns": [
            "Speed",
            "Design",
            "Price",
            "Battery life",
            "Camera quality"
          ],
          "rows": [
            "Phone A",
            "Phone B",
            "Phone C",
            "Phone D"
          ],
          "cellType": "checkbox"
        }
      ]
    }
  ]
}
```

## Live Demo

[View in Plunker](https://plnkr.co/edit/Jp9aukfXSkOR2Ee9)

## Learn More

Learn More

* [SurveyJS Documentation – onValidateQuestion event](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onValidateQuestion)
* [Matrix question – cellType: "checkbox"](https://surveyjs.io/form-library/examples/checkbox-matrix-question/)