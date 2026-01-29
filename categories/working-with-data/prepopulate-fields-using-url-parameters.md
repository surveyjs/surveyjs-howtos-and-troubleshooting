# Prepopulate Form Fields Using URL Parameters in SurveyJS

## Problem

My web page receives data via URL parameters, and I want to use this data to prepopulate one or more fields in a SurveyJS form.

## Solution

To prepopulate survey fields, read the URL parameters in JavaScript and assign their values to the corresponding survey questions by calling the `SurveyModel`'s [`setValue(questionName, value)`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#setValue) method. This approach allows you to initialize survey data programmatically before or after rendering the survey.

### Code Sample

The following example retrieves a support ticket number from the URL query string and assigns it to a question named `ticketNumber`:

```js
const params = new URLSearchParams(window.location.search);
const ticketNumber = params.get("ticketNumber");

// ...
// Omitted: `Survey.Model` creation
// ...

survey.setValue("ticketNumber", ticketNumber);
```

### Survey JSON Schema

```json
{
  "elements": [
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

With this setup, when the page is opened with a URL such as `?ticketNumber=12345`, the Ticket Number field is automatically filled with the provided value.