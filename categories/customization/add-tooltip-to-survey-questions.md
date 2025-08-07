# Add Tooltip Feature for Survey Questions

## Problem
How to add a tooltip feature with a toggle icon next to question titles that can be configured in SurveyJS Creator?

## Solution
In SurveyJS Surveys, use the question description field instead of tooltips. Tooltips are unavailable on mobile devices and provide poor accessibility. Use the description field which displays below the question title and is mobile-friendly:

```json
{
  "type": "text",
  "name": "question1",
  "title": "What is your name?",
  "description": "Please enter your full legal name as it appears on official documents"
}
```

However, if you need tooltip-like functionality, implement a custom [question title action](https://surveyjs.io/form-library/examples/modify-titles-of-survey-elements/) that displays a popup with additional information.

To enable tooltip text configuration in SurveyJS Creator, you can customize the property grid to allow users to specify tooltip content for questions. Implement question title actions which invoke a popup modal. For more information, refer to [Customize the Designer and Preview Tabs Demo](https://surveyjs.io/survey-creator/examples/survey-creator-custom-tabs/)
