# Bind Tagbox Values to a Dynamic Panel and Display Text Labels

## Problem
You want to bind a **Tagbox** question to a **Dynamic Panel**  so that each selected item generates a panel and a panel displays the selected item. However, instead of storing and displaying raw values (e.g., `"mother"`, `"father"`), you need the **Relationship** field inside each panel to show the corresponding **display text** (e.g., `"Mother"`, `"Father"`).

## Solution

Use a combination of:
- [`valueName`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-tag-box-model#valueName) to share data between the Tagbox and Dynamic Panel.
- [`valuePropertyName`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-tag-box-model#valuePropertyName) to store each selected value inside panel data.
- [`displayValue()`](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#displayvalue) function to convert stored values into human-readable text.

The key idea is:
- The Tagbox stores selected values in a shared array (`family`).
- Each dynamic panel instance gets one item from this array.
- Inside the panel, use `displayValue(questionName, value)` to resolve the display text. Use the `panel` prefix to reference the current dynamic panel's value in expressions.

In your case:
```js
"defaultValueExpression": "displayValue('relatives-source', {panel.selectedMember})"
```
Where:
* `relatives-source` is the name of a TagBox question.
* `{panel.selectedMember}` uses the TagBox'es value property name to access the `relatives-source` item corresponding to the current panel.


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
          "valueName": "family",
          "choices": [
            {
              "value": "mother",
              "text": "Mother"
            },
            {
              "value": "father",
              "text": "Father"
            },
            {
              "value": "stepmother",
              "text": "Stepmother"
            },
            {
              "value": "stepfather",
              "text": "Stepfather"
            },
            {
              "value": "brother",
              "text": "Brother"
            },
            {
              "value": "sister",
              "text": "Sister"
            },
            {
              "value": "half_sibling",
              "text": "Half-sibling"
            },
            {
              "value": "step_sibling",
              "text": "Step-sibling"
            },
            {
              "value": "son",
              "text": "Son"
            },
            {
              "value": "daughter",
              "text": "Daughter"
            },
            {
              "value": "stepchild",
              "text": "Stepchild"
            },
            {
              "value": "mother_in_law",
              "text": "Mother-in-law"
            },
            {
              "value": "father_in_law",
              "text": "Father-in-law"
            },
            {
              "value": "spouse",
              "text": "Spouse"
            },
            {
              "value": "partner",
              "text": "Partner"
            },
            {
              "value": "grandmother",
              "text": "Grandmother"
            },
            {
              "value": "grandfather",
              "text": "Grandfather"
            },
            {
              "value": "grandchild",
              "text": "Grandchild"
            },
            {
              "value": "aunt",
              "text": "Aunt"
            },
            {
              "value": "uncle",
              "text": "Uncle"
            },
            {
              "value": "cousin",
              "text": "Cousin"
            },
            {
              "value": "niece",
              "text": "Niece"
            },
            {
              "value": "nephew",
              "text": "Nephew"
            }
          ],
          "valuePropertyName": "selectedMember"
        },
        {
          "type": "paneldynamic",
          "name": "relative-details",
          "title": "Please provide the details of your immediate and extended family members.",
          "description": "Do not use initials",
          "valueName": "family",
          "bindings": {
            "panelCount": "family"
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
  ],
  "partialSendEnabled": true,
  "questionErrorLocation": "bottom",
  "progressBarShowPageTitles": true,
  "progressBarShowPageNumbers": true,
  "checkErrorsMode": "onValueChanged",
  "completeText": "Submit Form",
  "widthMode": "static",
  "width": "1200"
}
```

## Live Demo

[View in Plunker](https://plnkr.co/edit/p0v8N3VJ7bviFKQE)