# Create Dynamic Panels Based on User-Entered Number

## Problem

You need to collect repeated sets of information (for example, family member details), but the number of entries is unknown in advance. Instead of asking users to manually add panels, you want the survey to automatically generate the exact number of panels based on a numeric input provided by the user.

## Solution

Use a [Text (number)](https://surveyjs.io/form-library/examples/numeric-entry-question/) question to capture the desired number of items and bind it to a [Dynamic Panel](https://surveyjs.io/form-library/examples/questiontype-paneldynamic/) using the `bindings.panelCount` property.

When a respondent enters a number, the Dynamic Panel automatically creates that many panels. Each panel contains a predefined set of questions.

## Survey JSON Schema

```json
{
  "title": "Insurance Enrollment Form",
  "pages": [
    {
      "name": "part-c",
      "title": "Part C",
      "elements": [
        {
          "type": "text",
          "name": "relatives-source",
          "title": "How many family members do you wish to cover under your insurance?",
          "description": "You can include: parents, siblings, children (includes step siblings/children and/or half siblings) mother & father in-law).",
          "inputType": "number"
        },
        {
          "type": "paneldynamic",
          "name": "relative-details",
          "title": "Please provide the details of your immediate and extended family members.",
          "description": "Do not use initials",
          "bindings": {
            "panelCount": "relatives-source"
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
              "title": "Relationship"
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
          "addPanelText": "Add another relative"
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