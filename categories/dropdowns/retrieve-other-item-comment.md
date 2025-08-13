# Retrieve the Comment Value for the "Other" Option in a SurveyJS Dropdown

## Problem

When a Dropdown question has the "Other" option enabled ([`showOtherItem`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#showOtherItem) is set to `true`), I need to programmatically access the comment text entered by the user for that option.

## Solution

By default, the `SurveyModel`'s [`storeOthersAsComment`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#storeOthersAsComment) property is `true`. In this mode, the comment value for the "Other" option is stored separately in the [`data`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#data) object under a key formed as follows: `<question name> + <commentSuffix>`, where the [`commentSuffix`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#commentSuffix) defaults to `"-Comment"`. You can retrieve this comment value using the `SurveyModel`'s [`getValue()`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#getValue) method.

### Code Sample

```js
// ...
// Omitted: `Survey.Model` creation
// ...

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
          "choices": [ /* An array of choice options */ ],
          "showOtherItem": true
        }
      ]
    }
  ]
}
```