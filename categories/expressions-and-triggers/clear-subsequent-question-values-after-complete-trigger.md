# Clear Values of Subsequent Questions After a Complete Trigger

## Problem

In a multi-page survey, when a respondent navigates back and changes an answer that activates a Complete trigger, the values of subsequent questions remain in the survey data. This behavior can cause issues, as users typically expect all responses beyond the trigger point to be cleared once the survey is forced to complete.

## Solution

Use the [`onComplete`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onComplete) event to filter survey data. Iterate over the `SurveyModel`'s [`data`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#data) keys, keep values up to the trigger question, and stop once the trigger condition is met. The example below assumes `"continue"` is the trigger question.

### Survey JSON Schema

```javascript
{
  "pages": [
    {
      "name": "page1",
      "title": "Page 1",
      "elements": [
        {
          "type": "boolean",
          "name": "continue",
          "isRequired": true,
          "title": "Do you want to proceed with the survey?"
        }
      ]
    },
    {
      "name": "page2",
      "title": "Page 2",
      "elements": [
        {
          "type": "boolean",
          "name": "q2"
        }
      ]
    },
    {
      "name": "page3",
      "title": "Page 3",
      "elements": [
        {
          "type": "boolean",
          "name": "q3"
        }
      ]
    },
  ],
  "triggers": [
    {
      "type": "complete",
      "expression": "{continue} = false"
    }
  ],
  "headerView": "advanced"
}
```

### Code Sample

```javascript
// ...
// Omitted: `Survey.Model` creation
// ...

function clearAfterTrigger(data) {
  const filtered = {};
  for (const key of Object.keys(data)) {
    filtered[key] = data[key];
    if (key === "continue" && data[key] === false) {
      break;
    }
  }
  return filtered;
}

survey.onComplete.add(() => {
  const filteredData = clearAfterTrigger(survey.data);
  console.log(JSON.stringify(filteredData, null, 3)); // Save results here
});
```

## Learn More

- [Complete Trigger](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#complete)
- [Modify Survey Results](https://surveyjs.io/form-library/documentation/access-and-modify-survey-results#modify-survey-results)