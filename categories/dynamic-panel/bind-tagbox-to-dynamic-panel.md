# Bind Tag Box Values to a Dynamic Panel and Display Text Labels

## Problem

I want to bind a Multi-Select Dropdown (Tag Box) question to a Dynamic Panel so that each selected item creates a corresponding panel.

By default, the panel receives and displays the raw values (for example, `"mother"`, `"father"`). However, I need to display the user-friendly text labels instead (for example, `"Mother"`, `"Father"`).

## Solution

Use a combination of the following features:

- [`valueName`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-tag-box-model#valueName) to share data between the Tag Box and Dynamic Panel.
- [`valuePropertyName`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-tag-box-model#valuePropertyName) to store each selected item as an object field within panel data.
- [`displayValue()`](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#displayvalue) to resolve stored values into display text.

How it works:

1. Tag Box stores selected values in a shared array (`family`) via `valueName`.
2. `valuePropertyName` ensures each item is stored as an object (for example, `{ selectedMember: "mother" }`).
3. Dynamic Panel binds to the same array and creates one panel per item.
4. Inside each panel, `displayValue()` converts the stored value into its corresponding label.

### Survey JSON Schema

```json
{
  "title": "Background check consent form",
  "pages": [
    {
      "name": "part-c",
      "title": "Part C",
      "elements": [
        {
          "type": "tagbox",
          "name": "relatives-source",
          "title": "Select your immediate and extended family members.",
          "description": "Please include: parents, siblings, children (includes step siblings/children and/or half siblings) mother & father in-law).",
          "valueName": "family", // Share the data between Tag Box and Dynamic Panel
          "choices": [
            { "value": "mother", "text": "Mother" },
            { "value": "father", "text": "Father" },
            { "value": "stepmother", "text": "Stepmother" },
            { "value": "stepfather", "text": "Stepfather" },
            { "value": "brother", "text": "Brother" },
            { "value": "sister", "text": "Sister" },
            { "value": "half_sibling", "text": "Half-sibling" },
            { "value": "step_sibling", "text": "Step-sibling" },
            { "value": "son", "text": "Son" },
            { "value": "daughter", "text": "Daughter" },
            { "value": "stepchild", "text": "Stepchild" },
            { "value": "mother_in_law", "text": "Mother-in-law" },
            { "value": "father_in_law", "text": "Father-in-law" },
            { "value": "spouse", "text": "Spouse" },
            { "value": "partner", "text": "Partner" },
            { "value": "grandmother", "text": "Grandmother" },
            { "value": "grandfather", "text": "Grandfather" },
            { "value": "grandchild", "text": "Grandchild" },
            { "value": "aunt", "text": "Aunt" },
            { "value": "uncle", "text": "Uncle" },
            { "value": "cousin", "text": "Cousin" },
            { "value": "niece", "text": "Niece" },
            { "value": "nephew", "text": "Nephew" }
          ],
          "valuePropertyName": "selectedMember"
        },
        {
          "type": "paneldynamic",
          "name": "relative-details",
          "title": "Please provide the details of your immediate and extended family members.",
          "description": "Do not use initials",
          "valueName": "family", // Share the data between Tag Box and Dynamic Panel
          "bindings": {
            "panelCount": "family" // Generate a panel for each selected item
          },
          "templateElements": [           
            {
              "type": "text",
              "name": "relative-full-name",
              "title": "Full name",
              "description": "(surname and all given names, including maiden name)",
              "descriptionLocation": "underInput"
            },
            {
              "type": "text",
              "name": "relative-relationship",
              "startWithNewLine": false,
              "title": "Relationship",
              // Retrieve the user-friendly item label
              "defaultValueExpression": "displayValue('relatives-source', {panel.selectedMember})"
            },
            {
              "type": "text",
              "name": "relative-place-of-birth",
              "title": "Place of birth",
              "description": "(city, province or state, and country)\n",
              "descriptionLocation": "underInput"
            },
            {
              "type": "text",
              "name": "relative-date-of-birth",
              "startWithNewLine": false,
              "title": "Date of birth",
              "inputType": "date"
            },
            {
              "type": "comment",
              "name": "relative-current-address",
              "title": "Current address",
              "description": "(apt/street #, street name, city, province or state, and country)",
              "descriptionLocation": "underInput",
              "rows": 1,
              "autoGrow": true
            },
            {
              "type": "text",
              "name": "relative-date-of-death",
              "startWithNewLine": false,
              "title": "Date of death",
              "description": "(if applicable)",
              "descriptionLocation": "underInput",
              "inputType": "date"
            },
            {
              "type": "comment",
              "name": "relative-name-address-employer",
              "title": "Name and address of employer",
              "rows": 1,
              "autoGrow": true
            },
            {
              "type": "text",
              "name": "relative-job-title",
              "startWithNewLine": false,
              "title": "Job title"
            }
          ],
          "templateTitle": "Relative #{panelIndex}",
          "allowAddPanel": false,
          "allowRemovePanel": false,
          "panelsState": "firstExpanded",
          "addPanelText": "Add another relative",
          "displayMode": "tab"
        }
      ]
    }
  ]
}
```

## Live Demo

[Open in Plunker](https://plnkr.co/edit/p0v8N3VJ7bviFKQE)