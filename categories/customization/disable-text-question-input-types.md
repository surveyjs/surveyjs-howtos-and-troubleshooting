# Title
How to Remove Specific Input Types for Text Question Type in SurveyJS

## Problem
How to customize the input types available for a Text question type in SurveyJS. For instance, remove the "color" input type from the list of options.

## Solution
To restrict the available input types for a Text question type, use the `Serializer.findProperty("text", "inputType").setChoices()` function from the `survey-core` package. This allows you to define a custom list of input types, excluding unwanted options like `"color"`. You can apply the same approach to the Multiple Textboxes question type.

### Code Sample
```javascript
import { Serializer } from "survey-core";

// Limit input types for Text question type
Serializer.findProperty("text", "inputType").setChoices(["text", "date", "email", "number"]);

// Limit input types for Multiple Textboxes question type
Serializer.findProperty("multipletextitem", "inputType").setChoices(["text", "date", "email", "number"]);
```

### Survey JSON Schema
Below is an example of a Survey JSON schema that contains a Text question with a restricted input type (e.g., "email"). The `inputType` property is set to one of the allowed values defined in the code above.

```json
{
  "elements": [
    {
      "type": "text",
      "name": "userEmail",
      "title": "Enter your email address",
      "inputType": "email"
    }
  ]
}
```

## Learn More
- [SurveyJS Documentation: Serializer](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)
- [SurveyJS Text Question](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model)
- [SurveyJS Multiple Textboxes Question](https://surveyjs.io/form-library/documentation/api-reference/multiple-text-entry-question-model)