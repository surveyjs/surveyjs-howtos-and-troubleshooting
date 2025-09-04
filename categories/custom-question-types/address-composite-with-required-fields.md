# Create a Reusable Composite Question and Dynamically Make Nested Questions Required

## Problem

I want to create an "Address" composite question that represents a structured address form with fields like Street Address, City, State, and Zip Code. Additionally, I need the ability to control whether each nested field is required using custom properties.

## Solution

In SurveyJS, composite questions are reusable custom question types that combine existing questions into a single component. They are especially useful when you want to encapsulate a group of related fields (such as address information) into a single question that can be reused across surveys.

Composite questions output an object with nested values. For example:

```json
{
  "homeAddress": {
    "AddressLine1": "123 Main St",
    "City": "Anytown",
    // ... more nested question values
  }
}
```

To create a reusable "Address" composite question and make nested fields required dynamically, follow the steps below:

1. Register the component using the `ComponentCollection`.
2. Configure nested fields in the [`elementsJSON`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#elementsJSON) array.
3. Add custom Boolean properties (e.g., `requireAddressLine1`, `requireCity`) to control the `isRequired` property of each nested field. These properties can be overridden in the survey JSON schema or Property Grid. Use the [`onInit`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onInit) callback to add these properties.
4. Apply the custom properties initially using the [`onLoaded`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onLoaded) callback.
5. Use the [`onPropertyChanged`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onPropertyChanged) callback to update the `isRequired` property of the nested fields dynamically when custom properties change.
6. Add the composite question to your survey JSON schema and configure the custom properties as needed.

### Code Sample

```typescript
import { ComponentCollection, Serializer } from "survey-core";

function applyRequiredSettings(question) {
  const panel = question.contentPanel;
  if (!panel) return;

  panel.getQuestionByName("AddressLine1").isRequired = question.requireAddressLine1;
  panel.getQuestionByName("AddressLine2").isRequired = question.requireAddressLine2;
  panel.getQuestionByName("City").isRequired = question.requireCity;
  panel.getQuestionByName("State").isRequired = question.requireState;
  panel.getQuestionByName("ZipCode").isRequired = question.requireZipCode;
}

// Step 1: Register the component
ComponentCollection.Instance.add({
  name: "address",
  title: "Address",
  defaultQuestionTitle: "Enter your address:",
  // Step 2: Configure nested fields
  elementsJSON: [
    { type: "text", name: "AddressLine1", title: "Street Address", isRequired: true },
    { type: "text", name: "AddressLine2", title: "Street Address 2", startWithNewLine: false },
    { type: "text", name: "City", title: "City", isRequired: true },
    { type: "text", name: "State", title: "State", isRequired: true, startWithNewLine: false },
    { type: "text", name: "ZipCode", title: "Zip Code", isRequired: true, startWithNewLine: false }
  ],
  // Step 3: Add custom Boolean properties
  onInit() {
    Serializer.addProperty("address", { name: "requireAddressLine1:boolean", default: true, category: "general" });
    Serializer.addProperty("address", { name: "requireAddressLine2:boolean", default: false, category: "general" });
    Serializer.addProperty("address", { name: "requireCity:boolean", default: true, category: "general" });
    Serializer.addProperty("address", { name: "requireState:boolean", default: true, category: "general" });
    Serializer.addProperty("address", { name: "requireZipCode:boolean", default: true, category: "general" });
  },
  // Step 4: Apply the custom properties initially
  onLoaded(question) {
    applyRequiredSettings(question);
  },
  // Step 5: Update the nested fields when custom properties change
  onPropertyChanged(question, propertyName, value) {
    if (propertyName.startsWith("require")) {
      applyRequiredSettings(question);
    }
  }
});
```

### Survey JSON Schema

```json
{
  "elements": [
    {
      "type": "address",
      "name": "homeAddress",
      "title": "Home Address",
      "requireAddressLine2": true, // Make AddressLine2 required
      "requireState": false        // Make State optional
    }
  ]
}
```

## Learn More

- [Documentation: Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types)
- [Documentation: Add Custom Properties to Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)

<!-- ## Related Tags

- surveyjs
- composite-question
- custom-properties
- isRequired
- survey-json
- javascript -->