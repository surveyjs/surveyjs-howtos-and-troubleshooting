# Disable Editing, Copying, Deleting, and Changing Question Type on the Design Surface in Survey Creator

## Problem

I want to disallow the following actions on the design surface for specific questions:

- Editing the question's title and description
- Changing the question type
- Copying the question
- Deleting the question

At the same time, the Required toggle should still be able available.

## Solution

To control operations on a survey element, you can use the [`onElementAllowOperations`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onElementAllowOperations) event. This event provides Boolean flags (`allowDelete`, `allowCopy`, `allowChangeType`, etc.) to disable specific operations like changing the type or duplicating a question. However, `onElementAllowOperations` cannot be used to block only the title and description edits without affecting other editable properties&mdash;it would make the entire question read-only, preventing even the Required toggle.

To make only specific properties (e.g., `name`, `title`, `description`) read-only while keeping other functionality intact, handle the [`onPropertyGetReadOnly`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onPropertyGetReadOnly) event. This combination allows you to do the following:

- Restrict copy, delete, and type changes
- Lock selected properties
- Preserve the ability to toggle the required state

### Code Sample

```typescript
import { SurveyCreatorModel } from "survey-creator-core";
const creator = new SurveyCreatorModel();

// Questions to restrict (e.g., composite "Address" fields)
const addressFields = ["AddressLine1", "AddressLine2", "City", "State", "ZipCode"];

// Step 1: Restrict copying, deleting, and type changes
creator.onElementAllowOperations.add((_, options) => {
  const questionName = options.element.name;
  if (addressFields.includes(questionName)) {
    options.allowDelete = false;          // Hide Delete button
    options.allowCopy = false;            // Hide Copy button
    options.allowChangeType = false;      // Hide question type dropdown
    options.allowChangeInputType = false; // Hide input type dropdown
  }
});

// Step 2: Make name, title, and description read-only
creator.onPropertyGetReadOnly.add((_, options) => {
  const propertyList = ["name", "title", "description"];
  if (
    propertyList.includes(options.property.name) &&
    addressFields.includes(options.element.name)
  ) {
    options.readOnly = true;
  }
});
```

<!-- ## Related Tags
- surveyjs
- survey-creator
- question-editing
- isRequired
- custom-properties
- javascript -->
