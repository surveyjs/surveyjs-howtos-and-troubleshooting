# Enable Recursive Question Numbering in Custom Composite Questions

## Problem
When creating a custom [composite element](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types)), enabling survey-wide question numbering (`survey.showQuestionNumbers = "on"`) does not apply to questions inside the composite element. As a result, inner questions within the composite do not display numbers.

For example, if the composite element is numbered **2**, its inner questions should display as **2.1, 2.2, etc.**, but they remain unnumbered.

## Solution
To ensure that inner questions within a composite element inherit numbering, configure the composite question’s [`onCreated`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onCreated) function. In this function, access the `contentPanel` of the composite and set its [`showQuestionNumbers`](https://surveyjs.io/form-library/documentation/api-reference/panel-model#showQuestionNumbers) property to `"recursive"`.

### Code Sample
```js
import { ComponentCollection } from "survey-core";

ComponentCollection.Instance.add({
  name: "address",
  title: "Address",
  defaultQuestionTitle: "Enter your address:",
  elementsJSON: [
    {
      type: "text",
      name: "addressLine1",
      title: "Address",
      placeholder: "495 Grove Street",
    },
    {
      type: "text",
      name: "addressLine2",
      title: "Address line 2",
      placeholder: "Apartment #20",
    },
    {
      type: "text",
      name: "city",
      title: "City/Town",
      placeholder: "New York",
    },
    {
      type: "text",
      name: "state",
      startWithNewLine: false,
      title: "State/Region/Province",
      placeholder: "NY",
    },
    {
      type: "text",
      name: "postCode",
      title: "Zip/Postal code",
      inputType: "number",
      min: 501,
      max: 14925,
      placeholder: "10014",
      startWithNewLine: false,
    },
    {
      type: "dropdown",
      name: "country",
      title: "Country",
      placeholder: "USA",
      choicesByUrl: {
        url: "https://surveyjs.io/api/CountriesExample",
      },
    },
  ],
  onCreated: function (question) {
    question.contentPanel.showQuestionNumbers = "recursive";
  },
});
```

### Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "text",
          "name": "userName",
          "title": "User Name",
          "description": "Enter your full name here"
        },
        {
          "type": "address",
          "name": "question1"
        }
      ]
    }
  ],
  "showQuestionNumbers": "on"
}
```

## Learn More

* [SurveyJS – Create Custom Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types)
* [Question Auto-Numbering](https://surveyjs.io/form-library/examples/how-to-number-pages-and-questions/)