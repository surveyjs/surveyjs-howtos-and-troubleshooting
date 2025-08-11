# Create a Reusable Specialized Question and Place It in a Custom Toolbox Category

## Problem

I want to create a reusable custom question called "Location" that captures latitude and longitude. This question should appear in the Survey Creator's Toolbox under the "Text" category, alongside standard text questions like Single-Line Input, Long Text, and Multiple Textboxes.

## Solution

To achieve this, we'll define a specialized question type&mdash;a reusable component based on an existing question type, preconfigured with custom properties and content.

Follow these steps:

1. Register the new component using the `ComponentCollection`.
2. Define the base question structure using the `questionJSON` object. For "Location", we'll use the [Multiple Textboxes](https://surveyjs.io/form-library/examples/multiple-text-box-question/) question type.
3. Move the question to the desired toolbox category using the [`changeCategory()`](https://surveyjs.io/survey-creator/documentation/api-reference/questiontoolbox#changeCategory) method.

### Code Sample

```typescript
import { ComponentCollection } from "survey-core";
import { SurveyCreatorModel } from "survey-creator-core";

// Step 1: Register the Location component
ComponentCollection.Instance.add({
  name: "location",
  title: "Location",
  iconName: "icon-textedit-24x24", // Replace with a custom icon if needed
  defaultQuestionTitle: "Location",
  // Step 2: Define the base question structure
  questionJSON: {
    type: "multipletext",
    items: [
      { name: "lat", title: "Latitude" },
      { name: "lng", title: "Longitude" }
    ]
  }
});

const creator = new SurveyCreatorModel();
// Step 3: Move the question to the Text toolbox category
creator.toolbox.changeCategory("location", "text");
```

## Learn More

- [Documentation: Create Specialized Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-specialized-question-types)

<!-- ## Related Tags
- surveyjs
- custom-question
- toolbox
- multipletext
- survey-creator
- javascript -->
