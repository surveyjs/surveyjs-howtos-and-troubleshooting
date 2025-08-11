# Make All Cells in a Multi-Select Matrix Required

## Problem

I want to create a multi-select matrix where each cell must be answered.

## Solution

In a [Multi-Select Matrix](https://surveyjs.io/form-library/examples/multi-select-matrix-question/), you can require all cells by setting the [`isRequired`](https://surveyjs.io/form-library/documentation/api-reference/multi-select-matrix-column-values#isRequired) property to true for each column.

### Survey JSON Schema

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
              "name": "staffHelpfulness",
              "title": "Staff Helpfulness",
              "cellType": "rating",
              "rateType": "stars",
              "rateCount": 3,
              "rateMax": 3,
              "minRateDescription": "Very Poor",
              "maxRateDescription": "Excellent",
              "isRequired": true
            },
            {
              "name": "responseTime",
              "title": "Response Time",
              "cellType": "rating",
              "rateType": "stars",
              "rateCount": 3,
              "rateMax": 3,
              "minRateDescription": "Very Poor",
              "maxRateDescription": "Excellent",
              "isRequired": true
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

## Learn More

- [Multi-Select Matrix API Reference](https://surveyjs.io/form-library/documentation/api-reference/matrix-table-with-dropdown-list)
- [Multi-Select Matrix Demo](https://surveyjs.io/form-library/examples/multi-select-matrix-question/)