# Enable Recursive Question Numbering in Custom Composite Questions

## Problem

I created a [composite question type](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types) and enabled survey-wide numbering using the [`showQuestionNumbers`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#showQuestionNumbers) property. However, inner elements of the composite question are not numbered. I want them to be numbered recursively. For instance, if the composite question is number 2, nested questions should be numbered 2.1, 2.2, 2.3, and so on.

## Solution

To number elements inside a composite question, implement the question's [`onCreated`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onCreated) function. Within it, use the `contentPanel` property to access the panel that contains nested elements and set its `showQuestionNumbers` property to one of the following values:

- `"recursive"` - Continues numbering with a hierarchical pattern.
- `"onpanel"` - Restarts numbering within this composite element.
- `"default"` - Inherit that numbering pattern from a higher-level survey element.

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
  onCreated: (question) => {
    question.contentPanel.showQuestionNumbers = "recursive"; // or "default" | "onpanel"
  }
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
          "title": "User Name"
        },
        {
          "type": "address",
          "name": "question1"
        }
      ]
    }
  ],
  "showQuestionNumbers": true
}
```

## Learn More

- [Create Custom Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types)
- [Question Auto-Numbering Demo](https://surveyjs.io/form-library/examples/how-to-number-pages-and-questions/)