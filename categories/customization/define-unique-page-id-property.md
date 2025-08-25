# Add a Unique Page ID in SurveyJS

## Problem
How can I add a unique `id` property to the `pages` object in a SurveyJS form, alongside the existing `name` property, to uniquely identify each page?

## Solution
To add a unique page ID (`pid`) to the `pages` object in a SurveyJS form, you can use the `Serializer` API to register a custom `pid` property for the `"page"` class, as the built-in [`id`](https://surveyjs.io/form-library/documentation/api-reference/panelmodelbase#id) property is already defined in the `PanelModelBase` class. Additionally, you can implement event handlers to generate unique `pid` values automatically and ensure users specify unique values.

### Code Sample
```js
import { Serializer } from "survey-core";

// Register custom pid property for pages
Serializer.addProperty("page", {
  name: "pid:string",
  displayName: "Page ID",
  category: "general",
  isRequired: true,
});

// Generate unique pid for new pages
creator.onPageAdded.add((sender, options) => {
  const page = options.page;
  let index = 1;
  let pid = `p${index}`;
  const pages = sender.survey.pages;
  while (pages.some((p) => p["pid"] === pid)) {
    index++;
    pid = `p${index}`;
  }
  page["pid"] = pid;
});

// Validate uniqueness of pid
creator.onPropertyDisplayCustomError.add((sender, options) => {
  if (options.element.getType() === "page" && options.propertyName === "pid") {
    const newValue = options.value?.trim();
    if (!newValue) {
      options.error = "Page ID cannot be empty.";
      return;
    }
    const pages = sender.survey.pages;
    const duplicate = pages.find(
      (p) => p !== options.obj && p["pid"] === newValue
    );
    if (duplicate) {
      options.error = "This Page ID is already in use. Please choose a unique one.";
    }
  }
});
```

### Survey JSON Schema
Below is an example of how the `pages` object in the survey JSON will include the custom `pid` property:

```json
{
  "pages": [
    {
      "name": "page1",
      "pid": "p1",
      "elements": [
        {
          "type": "text",
          "name": "question1",
          "title": "Your name"
        }
      ]
    },
    {
      "name": "page2",
      "pid": "p2",
      "elements": [
        {
          "type": "text",
          "name": "question2",
          "title": "Your email"
        }
      ]
    }
  ]
}
```

## Learn More
- [SurveyJS PageModel Documentation](https://surveyjs.io/form-library/documentation/api-reference/page-model)
- [SurveyJS PanelModelBase Documentation](https://surveyjs.io/form-library/documentation/api-reference/panelmodelbase)
- [Add Custom Properties to Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)
- [Survey Creator `onPageAdded` Event](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPageAdded)
- [Survey Creator `onPropertyDisplayCustomError` Event](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyDisplayCustomError)