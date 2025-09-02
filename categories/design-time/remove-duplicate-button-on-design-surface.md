# Remove the Duplicate Button in SurveyJS Creator

## Problem

I want to hide the Duplicate button for pages, panels, and questions on the design surface in Survey Creator.

## Solution

In SurveyJS Creator, design surface controls are called **adorners**. To hide the Duplicate adorner, handle the [`onElementAllowOperations`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementAllowOperations) event. Inside the event handler, set the `options.allowCopy` property to `false`. 

You can use the same approach to remove other adorners, such as the input type dropdown or the drag handle.

### Code Sample

```javascript
// ...
// Omitted: `SurveyCreatorModel` creation
// ...

// Hide the Duplicate adorner for all survey elements
creator.onElementAllowOperations.add((_, options) => {
  options.allowCopy = false;
});
```

## Learn More

- [Specify Adorner Availability](https://surveyjs.io/survey-creator/documentation/customize-survey-creation-process#specify-adorner-availability)