# Disable In-Place Editing for Specific Question Elements in SurveyJS

## Problem

I want to prevent users from editing certain parts of a question&mdash;such as the title, description, or choice texts&mdash;directly on the design surface in Survey Creator.

## Solution

Use the [`onAllowInplaceEdit`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onAllowInplaceEdit) event to control which elements can be edited in-place. By inspecting the property being edited, you can selectively disable editing for titles, descriptions, or any other fields.

### Code Sample

```javascript
// Define the question elements you want to make read-only
const readOnlyInplaceEditors = ["title", "description", "text"];

// Add an event handler to disable inline editing for specific properties
creator.onAllowInplaceEdit.add((_, options) => {
  if (readOnlyInplaceEditors.indexOf(options.propertyName) >= 0) {
    options.allow = false;
  }
});
```

### Live Demo

[Open in CodeSandbox](https://codesandbox.io/p/sandbox/sad-keldysh-txr5pn?file=%2Fpackage.json%3A9%2C30)