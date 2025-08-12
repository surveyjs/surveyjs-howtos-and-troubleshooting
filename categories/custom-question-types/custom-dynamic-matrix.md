# Custom Dynamic Matrix Question Type

## Problem
How to create a specialized question type that copies the functionality and rendering of a Dynamic Matrix?

## Solution
To create a custom question type that replicates the functionality and rendering of a Dynamic Matrix question, you need to define a custom question model that extends the [`QuestionMatrixDynamicModel`](https://surveyjs.io/form-library/documentation/api-reference/dynamic-matrix-table-question-model) and register it with the `ElementFactory`. Use the `Serializer` to add the custom type and register the question with both the survey core and the UI framework (e.g., React).

This allows you to programmatically access a question using a unique type identifier and persist the Dynamic Matrix's behavior and appearance.

## Code Sample
```javascriptreact
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

// Register a custom question model
export class CustomDynamicMatrix extends QuestionMatrixDynamicModel {
  getType() {
    return CUSTOM_TYPE;
  }
  getCssType() {
    return "matrixdynamic";
  }
}
ElementFactory.Instance.registerElement(CUSTOM_TYPE, (name) => {
  return new CustomDynamicMatrix(name);
});

// Configure JSON Serialization
Serializer.addClass(
  CUSTOM_TYPE,
  [],
  () => {
    return new CustomDynamicMatrix("");
  },
  "matrixdynamic"
);

// Apply the default Dynamic Matrix renderer
ReactQuestionFactory.Instance.registerQuestion(CUSTOM_TYPE, function (props) {
  return React.createElement(SurveyQuestionMatrixDynamic, props);
});

// Specify the default configuration for a custom dynamic matrix
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

// Copy the toolbox icon for a custom question type
surveySettings.customIcons["icon-custom-matrix"] = "icon-matrixdynamic";
```

## Learn More
- [Integrate Third-Party React Components](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-react)
- [Add a Custom Question Type](https://surveyjs.io/form-library/examples/add-custom-question-type-to-form-builder/)
- [Implement a Descriptive Text Element](https://surveyjs.io/survey-creator/examples/custom-descriptive-text-element/)