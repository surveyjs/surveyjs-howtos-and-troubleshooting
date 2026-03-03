# Hide or Show a Question Based on Another Question's Visibility

## Problem

I want to hide `question1` when `question2` is hidden and show `question1` when `question2` is visible.

## Solution

You can access survey element properties directly in expressions and dynamic texts. This allows you to reference metadata such as an element's `visible` property. To control `question1` visibility based on `question2`, use the `visibleIf` property and reference `question2.visible` with the following syntax: `{$question2.visible}`. This expression evaluates to `true` or `false` depending on whether `question2` is currently visible.

### Survey JSON Schema

``` json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "boolean",
          "name": "visibilityToggle",
          "title": "Show Question 2",
          "description": "Turn this option on or off to show or hide Question 2",
          "defaultValue": false
        },
        {
          "type": "text",
          "name": "question2",
          "visibleIf": "{visibilityToggle} = true",
          "title": "Question 2"
        },
        {
          "type": "text",
          "name": "question1",
          "visibleIf": "{$question2.visible}",
          "title": "Question 1",
          "description": "Visible only when Question 2 is visible"
        }
      ]
    }
  ]
}
```

## Learn More

Refer to the SurveyJS documentation for more details about accessing element properties in expressions and dynamic texts: [Conditional Logic and Dynamic Texts | Element Properties](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#element-properties).
