# Remove Specific Input Types from a Text Input Question in SurveyJS

## Problem

When working in Survey Creator, I want to limit the available `inputType` options for a text input question. For example, I want to remove the Color input type from the list.

## Solution

The available input types are defined in the question's [`inputType`](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model#inputType) property. Access this property with the `Serializer`'s `findProperty` method and update its list of choices with the `setChoices()` method.

The example below retrieves the predefined `choices` array, filters out `"color"`, and updates the `inputType` property. The same approach works for both Single-Line Input and Multiple Textboxes questions.

### Code Sample

```javascript
import { Serializer } from "survey-core";

function filterChoices(qType, propName, blackList) {
  const prop = Serializer.findProperty(qType, propName);
  const newChoices = prop.choices.filter(choice => !blackList.includes(choice));
  prop.setChoices(newChoices);
}

// Remove "color" from the available input types
filterChoices("text", "inputType", [ "color" ]);
filterChoices("multipletextitem", "inputType", [ "color" ]);
```

## Learn More

- [Single-Line Input API Reference](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model)
- [Multiple Textboxes API Reference](https://surveyjs.io/form-library/documentation/api-reference/multiple-text-entry-question-model)