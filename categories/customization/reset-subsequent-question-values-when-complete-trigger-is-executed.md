# How to Clear all question values on pages which go after the page on which the survey was submitted by the Complete Trigger

## Problem
When navigating back to a previous page in a multi-page survey and changing an answer that triggers survey completion via the Complete trigger (e.g., setting `continue` to `false`), the values of subsequent questions (e.g., Question 2 and beyond) are not cleared. This is problematic in surveys with many questions (e.g., 100 questions), as users expect all subsequent question values to be reset upon survey completion.

## Solution
To address this, implement a function to programmatically clear all question values after the `continue` question when the survey is completed. This ensures that only relevant data is retained. The provided JavaScript function `trimAfterContinue` filters out all keys after the `continue` key when its value is `false`, and the `onComplete` event handler applies this logic.

### Code Sample
```javascript
function trimAfterContinue(obj) {
    const result = {};
    for (const key of Object.keys(obj)) {
      result[key] = obj[key];
      if (key === "continue" && obj[key] === false) {
        break;
      }
    }
    return result;
}

survey.onComplete.add((sender, options) => {
    const data = sender.data;
    const result = trimAfterContinue(data);
    console.log(JSON.stringify(result, null, 3));
});
```

### Survey JSON Schema
```javascript
{
  "pages": [
    {
      "name": "page0",
      "title": "Info",
      "elements": [
        {
          "type": "text",
          "name": "userName",
          "title": "User Name"
        }
      ]
    },
    {
      "name": "page1",
      "title": "Page 1",
      "elements": [
        {
          "type": "boolean",
          "name": "continue",
          "isRequired": true,
          "title": "Do you want to proceed filling in the survey?"
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
      "name": "page5",
      "title": "Page 3",
      "elements": [
        {
          "type": "boolean",
          "name": "q3",
          "title": "3"
        }
      ]
    },
    {
      "name": "page4",
      "title": "Page 4",
      "elements": [
        {
          "type": "boolean",
          "name": "q4",
          "title": "4"
        }
      ]
    },
    {
      "name": "page3",
      "title": "Page 5",
      "elements": [
        {
          "type": "boolean",
          "name": "q5",
          "title": "5"
        }
      ]
    }
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

## Learn More
- [Complete Trigger](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#complete)
- [Modify Survey Results](https://surveyjs.io/form-library/documentation/access-and-modify-survey-results#modify-survey-results)