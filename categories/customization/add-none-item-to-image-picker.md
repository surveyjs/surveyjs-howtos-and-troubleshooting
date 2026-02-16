# Add a "None of these" Option to an Image Picker in SurveyJS

## Problem

A [SurveyJS Image Picker](https://surveyjs.io/form-library/examples/image-picker-question/) question allows respondents to select one or multiple visual options. I need to add a "None of these" option that stores a distinct value in survey results, deselects all images when selected, and is automatically deselected when any image is selected.

## Solution

Implement mutual exclusivity between the Image Picker and a separate Checkboxes question as follows:

1. Add an Image Picker question to your survey.
2. Add a standalone Checkboxes question that contains a single custom choice: "None of these".
3. Configure the [`resetValueIf`](https://surveyjs.io/form-library/documentation/api-reference/question#resetValueIf) property on both questions so they clear each other's values.
4. (Optional) Place both questions inside a Panel to display them under a shared title and present them as a single logical block.

This approach keeps the data model explicit and avoids complex event handling.

## Survey JSON Schema

```json
{
  "elements": [
    {
      "type": "panel",
      "name": "panel1",
      "questionTitleLocation": "hidden",
      "title": "Select all logos that apply to a statement",
      "elements": [
        {
          "type": "imagepicker",
          "name": "logo",
          "resetValueIf": "{noLogoSelected} notempty",
          "choices": [
            {
              "value": "l1",
              "text": "Logo 1",
              "imageLink": "logo1.png"
            },
            {
              "value": "l2",
              "text": "Logo 2",
              "imageLink": "logo2.png"
            },
            {
              "value": "l3",
              "text": "Logo 3",
              "imageLink": "logo3.png"
            }
          ],
          "imageFit": "cover",
          "showLabel": true,
          "multiSelect": true
        },
        {
          "type": "checkbox",
          "name": "noLogoSelected",
          "resetValueIf": "{logo} notempty",
          "choices": [
            {
              "value": "noneOfThese",
              "text": "None of these"
            }
          ]
        }
      ]
    }
  ]
}
```

## Live Demo

[Open in Plunker](https://plnkr.co/edit/RTZ4q3pUzT1lz9qF)