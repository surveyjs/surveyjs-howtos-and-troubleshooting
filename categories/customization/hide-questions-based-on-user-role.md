# How to Show/Hide or Make Survey Questions Read-Only Based on User Role

## Problem
In many applications, you may want to customize a survey experience depending on the role of the person filling out the survey. For example, certain questions may only be visible to admins, or some fields may be read-only for regular users. The challenge is how to dynamically control question visibility or editability based on a user’s role within your system.

## Solution
SurveyJS provides two main ways to handle this:

1. **Using `visibleIf` and `enableIf` properties**  
   You can assign a conditio to a question's [`visibleIf`]() or [`enableIf`]() property to show, hide, or make questions read-only based on a survey [variable](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#variables) that stores the user's role. For more information on how to conditionally show questions, please refer to [Question Visibility](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#question-visibility).

2. **Using invisible Expression fields**  
   Store the user’s role in an invisible [Expression field](https://surveyjs.io/form-library/documentation/api-reference/expression-model) and reference it in `visibleIf` or `enableIf` conditions. Before showing the survey, set the expression field to the current user’s role using [`SurveyModel.setValue()`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#setValue).

## Code Example

### Use survey variables

```javascript
// Suppose you have a survey variable to store the user's role
survey.setVariable("userRole", currentUserRole);
```

Example of a question visible only to admins:

```json
{
  type: "text",
  name: "adminOnlyQuestion",
  title: "Admin Question",
  visibleIf: "{userRole} = 'admin'"
}
```

Example of a question read-only for non-admins:

```json
{
  type: "text",
  name: "editableQuestion",
  title: "Editable for admins",
  enableIf: "{userRole} = 'admin'"
}
```

Survey JSON Schema:

```json
{  
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "text",
          "name": "adminOnlyQuestion",
          "title": "Admin Question",
          "visibleIf": "{userRole} = 'admin'"
        },
        {
          "type": "text",
          "name": "editableQuestion",
          "title": "Editable for admins",
          "enableIf": "{userRole} = 'admin'"
        }
      ]
    }
  ]
}
```

### Use expression fields

Set a hidden Expression question value to store the user's role:
```js
survey.setValue("userRoleExpression", currentUserRole);
```

Example of a question visible only to admins:

```json
{
  "type": "text",
  "name": "adminOnlyQuestionExpression",
  "title": "Admin Question (Expression)",
  "visibleIf": "{userRoleExpression} = 'admin'"
}
```

Example of a question read-only for non-admins:

```json
{
  "type": "text",
  "name": "editableQuestionExpression",
  "title": "Editable for admins (Expression)",
  "enableIf": "{userRoleExpression} = 'admin'"
}
```


Survey JSON Schema:
```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "expression",
          "name": "userRoleExpression",
          "visible": false
        },
        {
          "type": "text",
          "name": "adminOnlyQuestionExpression",
          "title": "Admin Question (Expression)",
          "visibleIf": "{userRoleExpression} = 'admin'"
        },
        {
          "type": "text",
          "name": "editableQuestionExpression",
          "title": "Editable for admins (Expression)",
          "enableIf": "{userRoleExpression} = 'admin'"
        }
      ]
    }
  ]
}
```