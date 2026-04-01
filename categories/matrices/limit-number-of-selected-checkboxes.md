# Limit the Total Number of Selections in a Checkbox Matrix

## Problem

I am using a [Matrix with Checkboxes](https://surveyjs.io/form-library/examples/checkbox-matrix-question/) and need to restrict the total number of selected cells across the entire matrix (not per row or column, but globally).

## Solution

Handle validation in the [`onValidateQuestion`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onValidateQuestion) event. In the event handler, compute the total number of selected cells and assign a validation error if the limit is exceeded.

### Code Sample

```javascript
// Maximum allowed number of selected cells
const MAX_TOTAL_SELECTIONS = 4;

// ...
// Omitted: `Survey.Model` creation
// ...
survey.onValidateQuestion.add((_, options) => {
  // Replace with your matrix question name
  if (options.name !== "question1") return;

  const value = options.value || {};
  let totalSelected = 0;

  // Each key represents a row; each value is an array of selected column values
  Object.keys(value).forEach(rowKey => {
    const selectedInRow = value[rowKey];
    if (Array.isArray(selectedInRow)) {
      totalSelected += selectedInRow.length;
    }
  });

  if (totalSelected > MAX_TOTAL_SELECTIONS) {
     options.error =
      `You can select up to ${MAX_TOTAL_SELECTIONS} options in total. ` +
      `Currently selected: ${totalSelected}.`;
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

[Open in Plunker](https://plnkr.co/edit/Jt8Dx2vS24AD6OYi)