# Copy Selected Columns from One Dynamic Matrix to Another in SurveyJS

## Problem

You have two **dynamic matrices** in a SurveyJS form:

- **question1**: the source matrix with columns `a, b, c, d, ...`  
- **question2**: a target matrix inside a `paneldynamic` with columns `a1, a, b, c, d, ...`

**Goal:** Copy only certain columns (`a, b, c`) from `question1` to `question2` automatically whenever:

- A row is added/removed  
- A cell value changes  

**Why `setValueExpression` or `valueName` do not work:**

- `setValueExpression` updates the entire value of a target matrix. Therefore, `question2` receives a full copy of `question1`, rather than a subset of column values. As a result, user-entered values of other `question2` columns are reset, which is unwanted.  
- `valueName` synchronizes values of two matrices. However, the task is to copy values from source into target and not to share values between two matrices.  

## Solution

1. Add a **custom property** `mergeWithMatrix` to identify the source matrix.  
2. Track dependencies: any matrix using `mergeWithMatrix` should subscribe to source updates.  
3. Update the target matrix programmatically on:  
   - `onMatrixRowAdded`  
   - `onMatrixRowRemoved`  
   - `onMatrixCellValueChanging`
   
### Code Sample

```javascript
// 1. Add custom property to matrixdynamic
Survey.Serializer.addProperty("matrixdynamic", {
  name: "mergeWithMatrix:question", 
  category: "general"
});

// 2. Merge selected columns from source to destination
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

// 3. Track matrices dependent on a source matrix
function addToMatrixDependencies(src, dest) {
  if (!Array.isArray(src.matrixList)) src.matrixList = [];
  src.matrixList.push(dest);
}

function updateMatrixList(q) {
  if (q.isDescendantOf("matrixdynamic") && q.mergeWithMatrix) {
    const srcMatrix = survey.getQuestionByName(q.mergeWithMatrix);
    if (srcMatrix && srcMatrix.isDescendantOf("matrixdynamic")) {
      addToMatrixDependencies(srcMatrix, q);
      mergeMatrix(srcMatrix, q);
    }
  }
}

// 4. Get active dependent matrices
function getDependentMatrices(matrix) {
  const res = matrix.matrixList || [];
  return res.filter(m => !m.isDisposed);
}

// 5. Initialize existing matrices
survey.getAllQuestions(false, false, true).forEach(q => updateMatrixList(q));

// 6. Update matrices on creation
survey.onQuestionCreated.add((sender, options) => {
  updateMatrixList(options.question);
});

// 7. Sync on row addition/removal
survey.onMatrixRowAdded.add((sender, options) => {
  getDependentMatrices(options.question).forEach(m => m.addRow(false));
});
survey.onMatrixRowRemoved.add((sender, options) => {
  getDependentMatrices(options.question).forEach(m => m.removeRow(options.rowIndex, false));
});

// 8. Sync on cell value change
survey.onMatrixCellValueChanging.add((sender, options) => {
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
          "columns": [{"name":"a"},{"name":"b"},{"name":"c"},{"name":"x"},{"name":"y"},{"name":"z"}],
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
              "mergeWithMatrix": "question1",
              "columns": [{"name":"a1"},{"name":"a"},{"name":"b"},{"name":"c"},{"name":"d"},{"name":"e"}],
              "cellType": "text"
            }
          ]
        }
      ]
    }
  ]
}
```

## Learn More
* [`onMatrixCellValueChanging`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixCellValueChanging)
* [`onMatrixRowAdded`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixRowAdded)
* [`onMatrixRowRemoved`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onMatrixRowRemoved)
* [Custom Properties in SurveyJS](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)