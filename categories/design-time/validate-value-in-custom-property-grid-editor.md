# Register a Custom Property Grid Editor with Validation in SurveyJS Creator

## Problem

I have implemented a custom editor for a property in the Property Grid and need to add custom validation logic to it.

## Solution

In SurveyJS Creator, custom Property Grid editors are registered through `PropertyGridEditorCollection.register`. The configuration object passed to this method can include a `validateValue` callback that implements custom validation logic.

The `validateValue` function runs when the property value changes. If it returns `false`, the value is treated as invalid. To display a custom error message, call the `options.onPropertyDisplayCustomError` function.

### Code Sample

The example below registers a custom editor for properties of type `"shorttext"`. The editor renders a single-line text input limited to 15 characters, requires the value to contain at least 2 characters, and displays a custom validation message if the rule is violated.

```javascript
import { PropertyGridEditorCollection } from "survey-creator-core"; 

PropertyGridEditorCollection.register({
  // Apply this editor to properties of type "shorttext"
  fit: (prop) => prop.type === "shorttext",

  // Configure the underlying question JSON
  getJSON: () => {
    return { type: "text", maxLength: 15 };
  },

  // Custom validation logic
  validateValue: (obj, question, prop, val, options) => {
    // Let built-in required validation handle empty values
    if (val && val.length < 2) {
      options?.onPropertyDisplayCustomError?.(
        prop,
        "Please enter at least 2 characters"
      );
      return false; // Mark value as invalid
    }
    return true; // Value is valid
  },
});
```

To apply this editor, set `"shorttext"` as the property `type` when adding a new property or modifying an existing one via the `Serializer`.

```javascript
// Add a new custom property
Serializer.addProperty("question", {
  name: "shortname",
  displayName: "Short name",
  type: "shorttext",
  category: "general",
  visibleIndex: 3
});

// Change the editor for an existing property
Serializer.getProperty("question", "name").type = "shorttext";
```

## Learn More

- [Demo: Customize Property Editors](https://surveyjs.io/survey-creator/examples/customize-property-editors/)
- [`onPropertyDisplayCustomError` API Reference](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyDisplayCustomError)