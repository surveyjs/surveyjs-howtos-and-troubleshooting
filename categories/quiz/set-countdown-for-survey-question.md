# Configure Countdown Timer for Specific Survey Questions

## Problem
How to set a 60-second countdown timer for a specific survey question?

## Solution
To apply a countdown timer to a specific question, place the question on a separate page and set the [`PageModel.timeLimit`](https://surveyjs.io/form-library/documentation/api-reference/page-model#timeLimit) property to 60 seconds. When the time expires, the survey automatically advances to the next page. To hide the timer on pages without a time limit, use the [`survey.onCurrentPageChanged`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onCurrentPageChanged) and [`survey.onStarted`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onStarted) events to start or stop the timer based on the page's time limit. To prevent users from navigating backward, disable the Previous button with the [`survey.showPrevButton`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#showPrevButton) property.

## Code Sample
```javascript
// Survey JSON configuration
const surveyJson = {
  "title": "Sample Quiz",
  "showTimerPanel": "top",
  "showTimerPanelMode": "page",
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
      "maxTimeToFinish": 60,
      "elements": [
        {
          "type": "radiogroup",
          "name": "question2",
          "choices": ["Item 1", "Item 2", "Item 3"]
        }
      ]
    },
    {
      "name": "page3",
      "maxTimeToFinish": 0,
      "elements": [
        {
          "type": "radiogroup",
          "name": "question3",
          "choices": ["Item 1", "Item 2", "Item 3"]
        }
      ]
    },
    {
      "name": "page4",
      "maxTimeToFinish": 0,
      "elements": [
        {
          "type": "radiogroup",
          "name": "question4",
          "choices": ["Item 1", "Item 2", "Item 3"]
        }
      ]
    }
  ],
  "firstPageIsStarted": true
};

// JavaScript to control timer visibility and navigation
survey.showPrevButton = false;

function stopTimer(surveyModel) {
  surveyModel.stopTimer();
}

function startTimer(surveyModel) {
  surveyModel.startTimer();
}

survey.onCurrentPageChanged.add((sender, options) => {
  if (sender.currentPage.timeLimit === 0) {
    stopTimer(sender);
  } else {
    startTimer(sender);
  }
});

survey.onStarted.add((sender, options) => {
  if (sender.currentPage.timeLimit === 0) {
    stopTimer(sender);
  } else {
    startTimer(sender);
  }
});
```