# Create Address Composite Question in SurveyJS with Customizable Required Fields

## Question
How do I create an "address" composite question in SurveyJS that presents a structured address form (with fields like Street Address, City, State, Zip Code) and allows changing the `isRequired` property for the nested questions via custom properties?

### Base Address Elements JSON
```json
{
  "elements": [
    {
      "type": "text",
      "name": "AddressLine1",
      "title": "Street Address",
      "isRequired": true
    },
    {
      "type": "text",
      "name": "AddressLine2",
      "title": "Street Address 2",
      "startWithNewLine": false
    },
    {
      "type": "text",
      "name": "City",
      "title": "City",
      "isRequired": true
    },
    {
      "type": "text",
      "name": "State",
      "title": "State",
      "isRequired": true,
      "startWithNewLine": false
    },
    {
      "type": "text",
      "name": "ZipCode",
      "title": "Zip Code",
      "isRequired": true,
      "startWithNewLine": false
    }
  ]
}
```

## Answer
In SurveyJS, composite questions allow you to create reusable custom question types by combining existing questions into a single component. To create an "address" composite question and enable customization of the `isRequired` property for nested fields, register the component using `ComponentCollection`. Add custom boolean properties (e.g., `requireAddressLine1`) to the composite type via `Serializer.addProperty`. These properties default to match the base `elementsJSON` and can be overridden in the survey JSON or Property Grid.

Use callbacks like `onInit` to add properties, `onLoaded` to apply them initially, and `onPropertyChanged` to update dynamically when custom properties change.

### Step 1: Register the Composite Question
Import necessary modules and register the "address" component with the provided `elementsJSON`. Define a function to apply the `isRequired` settings based on custom properties.

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

ComponentCollection.Instance.add({
  name: "address",
  title: "Address",
  defaultQuestionTitle: "Enter your address:",
  elementsJSON: [
    { type: "text", name: "AddressLine1", title: "Street Address", isRequired: true },
    { type: "text", name: "AddressLine2", title: "Street Address 2", startWithNewLine: false },
    { type: "text", name: "City", title: "City", isRequired: true },
    { type: "text", name: "State", title: "State", isRequired: true, startWithNewLine: false },
    { type: "text", name: "ZipCode", title: "Zip Code", isRequired: true, startWithNewLine: false }
  ],
  onInit() {
    // Add custom properties for each nested field's isRequired (default matches elementsJSON)
    Serializer.addProperty("address", { name: "requireAddressLine1:boolean", default: true, category: "general" });
    Serializer.addProperty("address", { name: "requireAddressLine2:boolean", default: false, category: "general" });
    Serializer.addProperty("address", { name: "requireCity:boolean", default: true, category: "general" });
    Serializer.addProperty("address", { name: "requireState:boolean", default: true, category: "general" });
    Serializer.addProperty("address", { name: "requireZipCode:boolean", default: true, category: "general" });
  },
  onLoaded(question) {
    // Apply settings when the question is loaded
    applyRequiredSettings(question);
  },
  onPropertyChanged(question, propertyName, value) {
    // Re-apply if a require* property changes
    if (propertyName.startsWith("require")) {
      applyRequiredSettings(question);
    }
  }
});
```

### Step 2: Use the Composite Question in Your Survey
Once registered, add the "address" question to your survey JSON. Override custom properties as needed to change `isRequired` for nested fields.

```json
{
  "elements": [
    {
      "type": "address",
      "name": "homeAddress",
      "title": "Home Address",
      "requireAddressLine2": true,  // Make AddressLine2 required (overrides default false)
      "requireState": false        // Make State optional (overrides default true)
    }
  ]
}
```

**Explanation**:
- The composite question outputs an object with nested values, e.g., `{ "homeAddress": { "AddressLine1": "123 Main St", "City": "Anytown", ... } }`.
- Custom properties appear in the Property Grid for editing.
- The `applyRequiredSettings` function accesses nested questions via `contentPanel.getQuestionByName` and sets `isRequired` dynamically.

**Pros**: Reusable component; centralizes address logic; easy to customize required fields without duplicating JSON.  
**Cons**: Requires custom code for registration; changes only apply at runtime/loading.

## Recommended Approach
This method is ideal for forms needing standardized address inputs with flexible validation. Register the component once in your application code, then use it across surveys.

## Related Tags
- surveyjs
- composite-question
- custom-properties
- isRequired
- survey-json
- javascript

## References
- [SurveyJS Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types)
- [SurveyJS Add Custom Properties](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)
- [SurveyJS Serializer API](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)