# Copy Selected Columns Between Dynamic Matrices in SurveyJS

## Problem

I have two Dynamic Matrix questions in a SurveyJS form:

- `question1` &ndash; The source matrix with columns `a`, `b`, `c`, ...
- `question2` &ndash; The target matrix with columns `a1`, `a`, `b`, `c`, ...

I want to synchronize only specific columns (`a`, `b`, and `c`) from `question1` to `question2`. The synchronization should occur automatically when a row is added or removed or a cell value changes.

Standard approaches such as `setValueExpression` or `valueName` are not suitable here because they only support full-value binding, not selective column mapping.

## Solution

To implement this functionality, [add a custom property](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form) `mergeWithMatrix` that will identify the source matrix, establish a dependency between the source and target matrices by adding the targets to a list on the source matrix, and update the target matrix programmatically when the following events are raised: [`onMatrixRowAdded`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixRowAdded), [`onMatrixRowRemoved`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixRowRemoved), [`onMatrixCellValueChanging`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixCellValueChanging).

Implement a custom synchronization mechanism that:

1. Identifies a source matrix using a [custom property](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form) (`mergeWithMatrix`).
2. Tracks dependencies between matrices.
3. Synchronizes row structure and selected column values using SurveyJS events:
   - [`onMatrixRowAdded`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixRowAdded)
   - [`onMatrixRowRemoved`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixRowRemoved)
   - [`onMatrixCellValueChanging`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixCellValueChanging)

### Code Sample

```javascript
// 1. Add a custom property to identify the source matrix
Survey.Serializer.addProperty("matrixdynamic", {
  name: "mergeWithMatrix:question", 
  category: "general"
});

// 2. Copy selected columns from source to destination
function mergeMatrix(src, dest) {
  const srcValue = src.value;
  if (!Array.isArray(srcValue)) return;
  
  const destValue = dest.value;
  if (Array.isArray(destValue)) {
    for (let i = 0; i < Math.min(destValue.length, srcValue.length); i++) {
      const destRow = destValue[i];
      if (!!destRow) {
        ["a","b","c"].forEach(key => {
          if (key in srcValue[i]) destRow[key] = srcValue[i][key];
        });
      }
    }
  }
  dest.value = destValue;
}

// 3. Track dependent matrices on the source
function addToMatrixDependencies(src, dest) {
  if (!Array.isArray(src.matrixList)) src.matrixList = [];
  src.matrixList.push(dest);
}

// 4. Register dependencies and perform initial sync
function updateMatrixList(q) {
  if (q.isDescendantOf("matrixdynamic") && q.mergeWithMatrix) {
    const srcMatrix = survey.getQuestionByName(q.mergeWithMatrix);
    if (srcMatrix && srcMatrix.isDescendantOf("matrixdynamic")) {
      addToMatrixDependencies(srcMatrix, q);
      mergeMatrix(srcMatrix, q);
    }
  }
}

// 5. Get active dependent matrices
function getDependentMatrices(matrix) {
  const res = matrix.matrixList || [];
  return res.filter(m => !m.isDisposed);
}

// 6. Initialize existing matrices
survey.getAllQuestions(false, false, true).forEach(q => {
  updateMatrixList(q)
});

// 7. Handle dynamically created questions
survey.onQuestionCreated.add((_, options) => {
  updateMatrixList(options.question);
});

// 8. Synchronize row structure
survey.onMatrixRowAdded.add((_, options) => {
  getDependentMatrices(options.question).forEach(m => {
    m.addRow(false)
  });
});
survey.onMatrixRowRemoved.add((_, options) => {
  getDependentMatrices(options.question).forEach(m => {
    m.removeRow(options.rowIndex, false)
  });
});

// 9. Synchronize cell values
survey.onMatrixCellValueChanging.add((_, options) => {
  const rowIndex = options.row.rowIndex - 1;
  const colName = options.columnName;

  getDependentMatrices(options.question).forEach(m => {
    const rows = m.visibleRows;
    if (rowIndex < rows.length) {
      const row = rows[rowIndex];
      const cellQ = row.getQuestionByName(colName);
      if (cellQ && (cellQ.isEmpty() || cellQ.value === options.oldValue)) {
        cellQ.value = options.value;
      }
    }
  });
});
```

### Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "matrixdynamic",
          "name": "question1",
          "title": "Source Matrix",
          "columns": [
            { "name": "a" },
            { "name": "b" },
            { "name": "c" },
            { "name": "x" },
            { "name": "y" },
            { "name": "z" }
          ],
          "cellType": "text"
        },
        {
          "type": "paneldynamic",
          "name": "question3",
          "panelCount": 1,
          "templateElements": [
            {
              "type": "matrixdynamic",
              "name": "question2",
              "title": "Target Matrix",
              "mergeWithMatrix": "question1",
              "columns": [
                { "name": "a1" },
                { "name": "a" },
                { "name": "b" },
                { "name": "c" },
                { "name": "d" },
                { "name": "e" }
              ],
              "cellType": "text"
            }
          ]
        }
      ]
    }
  ]
}
```
