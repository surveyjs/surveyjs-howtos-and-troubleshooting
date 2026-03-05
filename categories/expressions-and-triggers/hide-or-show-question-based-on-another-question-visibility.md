# Hide or Show a Question Based on Another Question's Visibility

## Problem

I want to hide `question2` when `question1` is hidden and show `question2` when `question1` is visible.

## Solution

SurveyJS expressions and dynamic texts allow you to [access properties of survey elements](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#element-properties) directly. This means you can reference metadata such as an element's [`visible`](https://surveyjs.io/form-library/documentation/api-reference/question#visible) property in an expression.

To control the visibility of `question2` based on `question1`, use the [`visibleIf`](https://surveyjs.io/form-library/documentation/api-reference/question#visibleIf) property and reference the `visible` property of `question1` using the dollar sign (`$`) syntax: `{$question1.visible}`. This expression evaluates to `true` when `question1` is visible and `false` when it is hidden.

### Survey JSON Schema

``` json
{
  "elements": [
    {
      "type": "boolean",
      "name": "visibilityToggle",
      "title": "Show Question 1",
      "description": "Turn this toggle on or off to show or hide Question 2.",
      "defaultValue": false
    },
    {
      "type": "text",
      "name": "question1",
      "visibleIf": "{visibilityToggle} = true",
      "title": "Question 1"
    },
    {
      "type": "text",
      "name": "question2",
      "visibleIf": "{$question1.visible}",
      "title": "Question 2",
      "description": "Visible only when Question 1 is visible."
    }
  ]
}
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/KOX4fE5KkAZs65m8)

## Learn More

- [Conditional Logic and Dynamic Texts](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic)
