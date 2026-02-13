# Registering a Custom Property Grid Editor with Validation in SurveyJS Creator

## Problem

When building surveys using SurveyJS Creator, you might need a custom editor for a property in the Property Grid that enforces specific rules. For example:  

- Limit input length
- Prevent empty values if the property is required  
- Display a custom error message dynamically  

While SurveyJS automatically validates property values based on serialization metadata (e.g., `isRequired`), you may want to implement additional custom validation logic.

## Solution

SurveyJS allows you to register a custom property editor via `PropertyGridEditorCollection.register`. To apply custom validation to a property value, implement the `validateValue` function in your editor.

Here’s an example of a custom editor for properties of type `shorttext`, which limits input to 15 characters and shows an error message if the value is too short.

### Code Sample

```javascript
import { PropertyGridEditorCollection } from "survey-creator-core"; 

PropertyGridEditorCollection.register({
  // Apply this editor to properties of type "shorttext"
  fit: (prop) => prop.type === "shorttext",

  // Define a single-line text editor limited to 15 characters
  getJSON: () => {
    return { type: "text", maxLength: 15 };
  },

  // Custom validation logic
  validateValue(obj, question, prop, val, options) {
    // If value is empty but required, built-in serialization will handle it
    if (val && val.length < 2) {
      // Trigger a custom error message in the Property Grid
      options?.onPropertyDisplayCustomError?.(
        prop,
        "Please enter at least 2 characters"
      );
      return false; // mark as invalid
    }
    return true; // valid value
  },
});
```

To use a custom editor, set a property's `type` to a custom `"shorttext"` type. 
```
// Add a custom property that uses the "shorttext" editor
Serializer.addProperty("question", {
    name: "shortname",
    displayName: "Short name",
    type: "shorttext",
    category: "general",
    visibleIndex: 3
});

// Change the editor for the built-in `name` property
Serializer.getProperty("question", "name").type = "shorttext";
```

## See also

- [Demo: Customize Property Editors](https://surveyjs.io/survey-creator/examples/customize-property-editors/)
- [The `creator.onPropertyDisplayCustomError` function](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyDisplayCustomError)
