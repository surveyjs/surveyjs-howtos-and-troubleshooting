# Add a Unique Page ID in SurveyJS

## Problem

By default, survey pages only have a `name` property. However, I need an additional unique identifier (`id`) to reliably reference pages.

## Solution

Pages include a [built-in `id` property](https://surveyjs.io/form-library/documentation/api-reference/page-model#id), but it cannot be edited in Survey Creator. Instead, you can add a custom `pid` property to the `"page"` class using the `Serializer` API. To keep page IDs unique, generate them automatically in the [`onPageAdded`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPageAdded) event handler and validate user edits with the[`onPropertyDisplayCustomError`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyDisplayCustomError) event handler.

### Code Sample

```js
import { Serializer } from "survey-core";

// Add a custom `pid` property to pages
Serializer.addProperty("page", {
  name: "pid:string",
  displayName: "Page ID",
  category: "general",
  isRequired: true,
});

// ...
// Omitted: `SurveyCreatorModel` creation
// ...

// Auto-generate unique `pid` values for new pages
creator.onPageAdded.add((_, options) => {
  const page = options.page;
  let index = 1;
  let pid = `p${index}`;
  const pages = creator.survey.pages;
  while (pages.some((p) => p["pid"] === pid)) {
    index++;
    pid = `p${index}`;
  }
  page["pid"] = pid;
});

// Validate uniqueness of `pid`
creator.onPropertyDisplayCustomError.add((_, options) => {
  if (options.element.getType() === "page" && options.propertyName === "pid") {
    const newValue = options.value?.trim();
    if (!newValue) {
      options.error = "Page ID cannot be empty.";
      return;
    }
    const pages = creator.survey.pages;
    const duplicate = pages.find(
      (p) => p !== options.obj && p["pid"] === newValue
    );
    if (duplicate) {
      options.error = "This page ID is already in use. Enter a unique value.";
    }
  }
});
```

### Survey JSON Schema

Below is an example of how to specify the `pid` property in the survey JSON schema:

```json
{
  "pages": [
    {
      "name": "page1",
      "pid": "p1",
      "elements": [
        { "type": "text", "name": "question1" }
      ]
    },
    {
      "name": "page2",
      "pid": "p2",
      "elements": [
        { "type": "text", "name": "question2" }
      ]
    }
  ]
}
```

## Learn More

- [`PageModel`](https://surveyjs.io/form-library/documentation/api-reference/page-model)
- [`PanelModelBase`](https://surveyjs.io/form-library/documentation/api-reference/panelmodelbase)
- [Add Custom Properties to Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)