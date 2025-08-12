# Create a Custom Question Type by Duplicating a Built-In Question Type

## Problem

I want to create a custom question type that copies the functionality and rendering of another question type (for instance, Dynamic Matrix). I need this so that I can identify questions of this type and process its data differently from the parent question type.

## Solution

To create a custom question type that replicates the behavior and rendering of a built-in question, follow these steps:

1. Define a custom question model that extends the built-in question model (e.g., [`QuestionMatrixDynamicModel`](https://surveyjs.io/form-library/documentation/api-reference/dynamic-matrix-table-question-model) for Dynamic Matrix).
2. Register your question model in the `ElementFactory`.
3. Configure JSON serialization and deserialization for your question by calling the `Serializer.addClass` method.
4. Register the built-in Dynamic Matrix renderer (the `SurveyQuestionMatrixDynamic` class) as the renderer for your custom question in the `ReactQuestionFactory`.
5. Provide the default JSON configuration for your question type.
6. Specify the icon to use in the Survey Creator's Toolbox and other UI elements.

This approach lets you use your custom question type identifier (e.g., `"custom-matrix"`) in survey JSON schema while retaining the original Dynamic Matrix's behavior and look.

### Code Sample

```js
import { settings as creatorSettings } from "survey-creator-core";
import {
  QuestionMatrixDynamicModel,
  Serializer,
  ElementFactory,
  settings as surveySettings,
} from "survey-core";
import {
  SurveyQuestionMatrixDynamic,
  ReactQuestionFactory,
} from "survey-react-ui";

const CUSTOM_TYPE = "custom-matrix";

// Step 1: Define a custom question model
export class CustomDynamicMatrixModel extends QuestionMatrixDynamicModel {
  getType() {
    return CUSTOM_TYPE;
  }
  getCssType() {
    return "matrixdynamic";
  }
}

// Step 2: Register the custom question model
ElementFactory.Instance.registerElement(CUSTOM_TYPE, (name) => {
  return new CustomDynamicMatrixModel(name);
});

// Step 3: Configure JSON serialization and deserialization
Serializer.addClass(
  CUSTOM_TYPE,
  [],
  () => {
    return new CustomDynamicMatrixModel("");
  },
  "matrixdynamic"
);

// Step 4: Register the built-in Dynamic Matrix renderer for the custom question
ReactQuestionFactory.Instance.registerQuestion(CUSTOM_TYPE, function (props) {
  return React.createElement(SurveyQuestionMatrixDynamic, props);
});

// Step 5: Provide the default JSON configuration
creatorSettings.toolbox.defaultJSON[CUSTOM_TYPE] = {
  columns: [
    {
      name: "Column 1",
    },
    {
      name: "Column 2",
    },
    {
      name: "Column 3",
    },
  ],
  choices: [1, 2, 3, 4, 5],
};

// Step 6: Specify the icon to use in the Survey Creator UI
surveySettings.customIcons["icon-custom-matrix"] = "icon-matrixdynamic";
```

## Learn More

- [Integrate Third-Party React Components](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-react)
- [Add a Custom Question Type](https://surveyjs.io/form-library/examples/add-custom-question-type-to-form-builder/)
- [Implement a Descriptive Text Element](https://surveyjs.io/survey-creator/examples/custom-descriptive-text-element/)