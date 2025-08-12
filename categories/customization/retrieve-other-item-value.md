# How to Retrieve the Comment Value for the "Other" Option in a SurveyJS Dropdown 

## Problem
When a Dropdown question has the "Other" option enabled ([`showOtherItem`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#showOtherItem)), how to programmatically retrieve the comment value entered by the user for the "Other" option.

## Solution
When the [`survey.storeOthersAsComment`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#storeOthersAsComment) property is set to its default value (`true`), the comment for the "Other" option is stored in a separate  property within [`survey.data`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#data). 
This property is named by combining the [Dropdown](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model) question's name with the [`commentSuffix`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#commentSuffix) (typically `"-Comment"`). You can retrieve this comment value using the [`survey.getValue()`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#getValue) function by referencing the combined name.

### Code Sample
```js
survey.data = {
  "dropdown-question": "other",
  "dropdown-question-Comment": "User's custom input"
};

const dropdownQuestion = survey.getQuestionByName('dropdown-question');
const otherValue = survey.getValue(dropdownQuestion.name + survey.commentSuffix);
console.log(otherValue); // Outputs: "User's custom input"
```

### Survey JSON Schema
```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "dropdown",
          "name": "dropdown-question",
          "choices": [
            "Item 1",
            "Item 2",
            "Item 3"
          ],
          "showOtherItem": true
        }
      ]
    }
  ]
}
```