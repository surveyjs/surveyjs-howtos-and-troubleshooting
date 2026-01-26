# Disable Bulk Choice Edit in SurveyJS Creator

## Problem

In Survey Creator, the Choices table includes a bulk edit feature that allows users to modify multiple choices at once. I want to disable this functionality so that editing is restricted to individual items only.

## Solution

Use the [`onConfigureTablePropertyEditor`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onConfigureTablePropertyEditor) event to configure property editors for table-like properties such as choices. By setting the `options.allowBatchEdit` property to `false`, you can disable bulk editing.

### Code Sample

```js
// ...
// Omitted: `SurveyCreatorModel` creation
// ...
creator.onConfigureTablePropertyEditor.add((_, options) => {
  if (options.propertyName === "choices") {
    options.allowBatchEdit = false;
  }
});
```