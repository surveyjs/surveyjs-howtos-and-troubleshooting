Reset matrix row value when a row becomes disabled (enableIf)
=======================================================================

Problem
-------
When a matrix row becomes disabled using the row `enableIf` property, SurveyJS does not automatically clear the answer stored for that row. You may need to reset the value for that row (remove it from `matrix.value`) when the row is disabled so the disabled row's answer is not submitted with the survey.

Solution
--------
Listen to `survey.onPropertyValueChangedCallback` and, when a row's `isEnabled` changes to `false`, remove that row's value from the parent matrix `value` object.

Live example
------------
Plunker: https://plnkr.co/edit/YBFB6yw24zJh7VsY

Code
----
Insert this code after you create the survey (for example, right after `new Survey.Model(...)`):

```javascript
survey.onPropertyValueChangedCallback = (
  name,
  oldValue,
  newValue,
  sender,
  arrayChanges
) => {
  // When a row's enabled state becomes false, remove its value from the matrix
  if (name === "isEnabled" && newValue === false) {
    let matrix = sender.locOwner;
    // Ensure we operate on a matrix question and it has values
    if (!!matrix && matrix.getType() === "matrix" && !matrix.isEmpty()) {
      const val = matrix.value;
      // sender.value is the row key (row name). Delete that property from the matrix values.
      delete val[sender.value];
      // Reassign to trigger change notifications
      matrix.value = val;
    }
  }
};
```

How it works
------------
- The callback `onPropertyValueChangedCallback` fires when any property of SurveyJS objects changes.
- When a row's `isEnabled` changes and becomes `false`, `sender
