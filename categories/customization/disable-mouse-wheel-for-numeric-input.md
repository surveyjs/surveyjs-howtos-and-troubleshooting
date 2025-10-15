# Disable Mouse Wheel for Numeric Input in SurveyJS

## Problem

In a SurveyJS form, a Single-Line Input question with `inputType: "number"` lets users change the numeric value by scrolling the mouse wheel. This behavior can lead to accidental value changes, especially when scrolling through a page.

## Solution

A SurveyJS Single-Line Input question with `inputType: "number"` is rendered as a standard HTML `<input type="number">` element. By default, such elements allow users to scroll the mouse wheel to increment or decrement their value, typically in steps of 1 (or as defined by the `step` attribute).

To disable this behavior in a SurveyJS form, handle the [`onAfterRenderQuestion`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onAfterRenderQuestion) event and intercept the wheel event. For example, you can remove focus from the input when the wheel event occurs, allowing the page to scroll normally instead:

```javascript
survey.onAfterRenderQuestion.add((_, options) => {
  if (options.question.inputType === "number") {
    const inputField = options.htmlElement.querySelector("input");
    inputField.addEventListener('wheel', (event) => {
      inputField.blur();
    });
  }
});
```

Alternatively, you can avoid this issue entirely by using a text input with a numeric input mask instead of `inputType: "number"`. This approach preserves numeric validation and formatting without enabling the default wheel behavior. See the [Masked Input Fields](https://surveyjs.io/form-library/examples/masked-input-fields/) demo for an example.