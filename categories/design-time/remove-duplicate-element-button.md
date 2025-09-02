# Remove the Default "Duplicate" Button in SurveyJS Creator

## Problem
When using **SurveyJS Creator V2**, the default element action panel displays a **"Duplicate" button** for pages, panels, and questions.  
We want to **remove this button** from all element types.

## Solution
To remove the **Duplicate** adorner action, use the [`creator.onElementAllowOperations`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementAllowOperations) event. Within this event, set `options.allowCopy = false` . The "Duplicate" action will no longer appear in the action panel.

### Code Sample
```javascript
// Disable the "Duplicate" button globally for all elements
creator.onElementAllowOperations.add((sender, options) => {
  options.allowCopy = false;
});

## Learn More

[Specify Adorner Availability](https://surveyjs.io/survey-creator/documentation/customize-survey-creation-process#specify-adorner-availability)