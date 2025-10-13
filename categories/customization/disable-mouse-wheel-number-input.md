# Disable Mouse Wheel Scrolling for Numeric Input in SurveyJS

## Problem
For a SurveyJS Text question with `inputType: "number"`, scrolling the mouse wheel over a focused single-line numeric input field changes the number value. This behavior is due to the default functionality of the HTML `<input type="number">` element.

```json
{
  "type": "text",
  "name": "question1",
  "inputType": "number"
}
```

Is it possible to disable the default mouse wheel event for this numeric input type?

## Solution
The SurveyJS Text question with `inputType: "number"` uses a standard HTML `<input type="number">` element, which by default allows the mouse wheel to increment or decrement the value based on the `step` attribute (defaulting to 1 if unspecified). To disable this behavior, you can use the [`survey.onAfterRenderQuestion`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onAfterRenderQuestion) event to add an event listener that prevents the mouse wheel from modifying the input value. One effective approach is to make the input lose focus when the wheel event occurs, allowing the page to scroll instead.

### Code Sample
```javascript
survey.onAfterRenderQuestion.add((sender, options) => {
  if (options.question.inputType === "number") {
    const numericInput = options.htmlElement.getElementsByTagName("input")[0];
    numericInput.addEventListener('wheel', function(event) {
      numericInput.blur();
    });
  }
});
```

This code checks if the question's `inputType` is `"number"`, retrieves the input element, and adds a `wheel` event listener that triggers a `blur()` to remove focus from the input when the mouse wheel is used. This prevents the value from changing and allows normal page scrolling.

Alternatively, instead of using `inputType: "number"`, you can apply a Numeric input mask to a Text question to achieve similar numeric input functionality without the default wheel behavior. Refer to the [Masked Input Fields Demo](https://surveyjs.io/form-library/examples/masked-input-fields/) for more details.