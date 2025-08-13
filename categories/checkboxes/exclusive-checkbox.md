# Configure an Exclusive Checkbox Option in SurveyJS

## Problem
You have a checkbox question with multiple selectable items. `"Item 6"` should be an exclusive option. When "Item 6" is selected, all other options should be deselected, and when any other option is selected, `"Item 6"` should be deselected.

## Solution
SurveyJS provides a built-in `isExclusive` property for checkbox question choices. By setting `"isExclusive": true` on the `"Item 6"` choice, SurveyJS automatically handles the exclusivity logic, ensuring that selecting "Item 6" clears all other selected options and vice versa.

## Code Sample
Below is the SurveyJS JSON schema to configure a checkbox question with `"Item 6"` as an exclusive option.

### Survey JSON Schema
```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "checkbox",
          "name": "question43",
          "title": "Select options",
          "choices": [
            { "value": "item1", "text": "Item 1" },
            { "value": "item2", "text": "Item 2" },
            { "value": "item3", "text": "Item 3" },
            { "value": "item4", "text": "Item 4" },
            { "value": "item5", "text": "Item 5" },
            {
              "value": "item6",
              "text": "Item 6 (exclusive)",
              "isExclusive": true
            }
          ]
        }
      ]
    }
  ]
}
```

## Learn More
- **SurveyJS Checkbox Question Documentation**: Learn more about configuring checkbox questions at https://surveyjs.io/form-library/documentation/api-reference/checkbox-question-model.