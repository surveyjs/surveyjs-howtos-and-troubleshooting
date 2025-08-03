# Restrict Editing of Question Title, Description, Copying, Deleting, and Type in SurveyJS Creator designer

## Question
How can I configure SurveyJS Creator to disable editing the title, description, copying, deleting, and changing the type for specific questions (e.g., address-related fields), while still allowing the `isRequired` property to be toggled in the Property Grid?

## Answer
In SurveyJS Creator, you cannot use `options.allowEdit` in the `onElementAllowOperations` event to disable specific editing actions because it disables all adorners, including the `isRequired` toggle. Instead, use the `onElementAllowOperations` event to block copying, deleting, and type changes, and the `onPropertyGetReadOnly` event to make properties like `name`, `title`, and `description` read-only for specific questions. This preserves the ability to toggle `isRequired` in the Property Grid.

### Implementation
Define a list of question names (e.g., address fields) and use event handlers to restrict operations and make properties read-only.

```typescript
import { SurveyCreator } from "survey-creator-core";

// Initialize Survey Creator
const creator = new SurveyCreator();

// List of questions to restrict
const addressFields = ["AddressLine1", "AddressLine2", "City", "State", "ZipCode"];

// Restrict copying, deleting, and type changes
creator.onElementAllowOperations.add((sender, options) => {
  const questionName = options.element.name;
  if (addressFields.includes(questionName)) {
    options.allowDelete = false;         // Disable delete button
    options.allowChangeType = false;    // Disable type dropdown
    options.allowChangeInputType = false; // Disable input type changes
    options.allowCopy = false;          // Disable copy/duplication
  }
});

// Make name, title, description read-only
creator.onPropertyGetReadOnly.add((sender, options) => {
  if (
    ["name", "title", "description"].includes(options.property.name) &&
    addressFields.includes(options.element.name)
  ) {
    options.readOnly = true; // Prevent editing in Property Grid
  }
});
```

### How It Works
- **onElementAllowOperations**: Controls Creator UI actions like deleting, copying, or changing question types. Setting `allowDelete`, `allowChangeType`, `allowChangeInputType`, and `allowCopy` to `false` removes these options for specified questions.
- **onPropertyGetReadOnly**: Makes `name`, `title`, and `description` properties read-only in the Property Grid for questions in `addressFields`. The `isRequired` property remains editable since it’s not included in the read-only list.
- **Scope**: Applies only to questions with names in `addressFields` (e.g., `AddressLine1`, `City`). Adjust the array to target other questions.

**Pros**: Precise control over Creator UI and Property Grid; preserves `isRequired` toggling; reusable for any question.  
**Cons**: Requires custom code; must maintain the `addressFields` list.

### Example Usage
Use this in a survey with address fields, such as:
```json
{
  "elements": [
    { "type": "text", "name": "AddressLine1", "title": "Street Address", "isRequired": true },
    { "type": "text", "name": "City", "title": "City" }
  ]
}
```
In Creator, users can toggle `isRequired` for `AddressLine1` or `City` but cannot edit their titles, descriptions, names, copy, delete, or change their types.

## Recommended Approach
This solution is ideal for scenarios where specific questions (e.g., in a composite address question) need locked properties while allowing validation flexibility. Implement the code during Creator initialization.

## Related Tags
- surveyjs
- survey-creator
- question-editing
- isRequired
- custom-properties
- javascript
