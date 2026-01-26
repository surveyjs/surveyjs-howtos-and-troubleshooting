# Show, Hide, or Make Survey Questions Read-Only Based on User Role

## Problem

I want to customize the survey experience based on the role of the respondent. For example, certain questions should only be visible to administrators, or some fields should be read-only for regular users. How can I dynamically control question visibility or editability based on a user's role within my system?

## Solution

Define a survey variable that stores the user role, and use this variable in expressions assigned to the [`visibleIf`](https://surveyjs.io/form-library/documentation/api-reference/question#visibleIf) or [`enableIf`](https://surveyjs.io/form-library/documentation/api-reference/question#enableIf) properties of the questions you want to show, hide, or make read-only.

### Code Sample

The following code defines a `userRole` survey variable and sets its value to `"admin"`:

```js
// ...
// Omitted: `Survey.Model` creation
// ...
survey.setVariable("userRole", "admin");
```
### Survey JSON Schema:

The following survey JSON schema includes a question that is visible only to administrators and a question that is editable only for administrators and read-only for all other users.

```json
{
  "elements": [
    {
      "type": "text",
      "name": "adminOnlyQuestion",
      "title": "Visible to administrators",
      "visibleIf": "{userRole} = 'admin'"
    },
    {
      "type": "text",
      "name": "editableQuestion",
      "title": "Editable for administrators",
      "enableIf": "{userRole} = 'admin'"
    }
  ]
}
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/nnsY9tm4eidVhYCI)