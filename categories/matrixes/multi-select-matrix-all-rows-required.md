# Make All Rows Required for a Multi-Select Matrix in SurveyJS

## Problem
How to create a multi-select matrix question where each row is required to be answered?

## Solution
To ensure that respondents answer all rows in a [Multi-Select Matrix](https://surveyjs.io/form-library/examples/questiontype-matrixdropdown/), enable the [`MatrixDropdownColumn.isRequired`](https://surveyjs.io/form-library/documentation/api-reference/multi-select-matrix-column-values#isRequired) option for each matrix column.

## Code Sample
```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "matrixdropdown",
          "name": "serviceSatisfaction",
          "title": "Please rate your satisfaction with the following aspects in each department:",
          "columns": [
            {
              "name": "Staff Helpfulness",
              "title": "Staff Helpfulness",
              "cellType": "rating",
              "isRequired": true,
              "rateType": "stars",
              "rateCount": 3,
              "rateMax": 3,
              "minRateDescription": "Very Poor",
              "maxRateDescription": "Excellent"
            },
            {
              "name": "Response Time",
              "title": "Response Time",
              "cellType": "rating",
              "isRequired": true,
              "rateType": "stars",
              "rateCount": 3,
              "rateMax": 3,
              "minRateDescription": "Very Poor",
              "maxRateDescription": "Excellent"
            }
          ],
          "rows": [
            {
              "value": "salesDept",
              "text": "Sales Department"
            },
            {
              "value": "supportDept",
              "text": "Support Department"
            }
          ]
        }
      ]
    }
  ]
}
```