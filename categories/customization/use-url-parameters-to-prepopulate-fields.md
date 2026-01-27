# Pre-populate SurveyJS Form Fields Using URL Parameters

## Problem

How to supply data in the URL as parameters and pre-populate fields in a SurveyJS form?

For example, you may want to pass a ticket number from an external ticketing system and display it in a read-only field. This value can then be used to programmatically load or calculate additional data while the user fills out the form.


## Solution

To achieve this, retrieve URL parameters in JavaScript and assign them to a SurveyJS survey instance.

- Use [`survey.setVariable(name, value)`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#setVariable) to store values as [survey variables](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#variables), which can be referenced in expressions, visibility rules, calculated values, and dynamic texts. Survey variables are useful when you don’t want to bind URL parameters directly to a visible question or when the value is only needed for logic.
- Use [`survey.setValue(questionName, value)`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#setValue) to assign values directly to **questions**.

### Code Sample

```js
const survey = new Survey.Model(surveyJson);

// Read URL parameters
const params = new URLSearchParams(window.location.search);
const ticketNumber = params.get("ticketNumber");

// Store as a survey variable
survey.setVariable("ticketNumber", ticketNumber);

// Optionally assign to a question
survey.setValue("ticketNumber", ticketNumber);

survey.render(document.getElementById("surveyContainer"));
```

### Survey JSON Schema

```json
{
  "questions": [
    {
      "type": "text",
      "name": "ticketNumber",
      "title": "Ticket Number",
      "readOnly": true
    },
    {
      "type": "comment",
      "name": "issueDescription",
      "title": "Describe your issue"
    }
  ]
}
```

## Learn More

[Conditional Logic and Dynamic Texts | Variables](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic)