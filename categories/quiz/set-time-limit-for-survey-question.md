# Configure Time Limit for an Individual Survey Question

## Problem

I want to set a 60-second time limit for a single survey question.

## Solution

In SurveyJS Form Library, you can apply time limits to individual pages or the entire survey. To limit time for a specific question, place that question on its own page and apply a time limit to that page as follows:

1. Enable and display the page timer by setting the [`showTimer`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#showTimer) property to `true` and [`timerInfoMode`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#timerInfoMode) to `"page"`.
2. Isolate the question by placing it on a separate page and setting that page's [`timeLimit`](https://surveyjs.io/form-library/documentation/api-reference/page-model#timeLimit) property.
3. Disable backward navigation by setting [`showPrevButton`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#showPrevButton) to `false`.
4. Hide the timer on untimed pages by calling the [`startTimer()`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#startTimer) and [`stopTimer()`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#stopTimer) method within the [`onStarted`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onStarted) and [`onCurrentPageChanged`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onCurrentPageChanged) event handlers.

## Survey JSON Schema

```json
{
  "title": "Sample Quiz",
  // Step 1: Enable and display the page timer
  "showTimer": true,
  "timerInfoMode": "page",
  "firstPageIsStartPage": true,
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "html",
          "name": "question1",
          "html": "Now, we are ready to start!"
        }
      ]
    },
    {
      "name": "page2",
      "title": "Questions without time limit",
      "elements": [
        {
          "type": "radiogroup",
          "name": "question2",
          "choices": ["Item 1", "Item 2", "Item 3"]
        },
        {
          "type": "radiogroup",
          "name": "question3",
          "choices": ["Item 1", "Item 2", "Item 3"]
        }
      ]
    },
    // Step 2: Place the timed question on a separate page
    {
      "name": "page3",
      "title": "Question with limited time",
      "timeLimit": 60,
      "elements": [
        {
          "type": "radiogroup",
          "name": "question4",
          "choices": ["Item 1", "Item 2", "Item 3"]
        }
      ]
    }
  ]
}
```

## Code Sample

```js
// ...
// Omitted: `Survey.Model` creation
// ...

function toggleTimer(survey) {
  survey.currentPage.timeLimit === 0 ? survey.stopTimer() : survey.startTimer();
}

// Step 3: Disable backward navigation
survey.showPrevButton = false;

// Step 4: Hide the timer on untimed pages
survey.onCurrentPageChanged.add(toggleTimer);
survey.onStarted.add(toggleTimer);
```

## Learn More

- [Documentation: Create a Quiz](https://surveyjs.io/form-library/documentation/design-survey/create-a-quiz)
- [Demo: Quiz with Instant Results](https://surveyjs.io/form-library/examples/create-quiz-with-immediate-results/)
- [Demo: Review Quiz Results](https://surveyjs.io/form-library/examples/review-mode-for-quiz-results/)
- [Demo: Scored Quiz](https://surveyjs.io/form-library/examples/create-a-scored-quiz/)