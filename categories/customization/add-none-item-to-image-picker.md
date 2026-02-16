# Add a “None of these” Option to an Image Picker (Exclusive Selection)

## Problem

When using an [Image Picker](https://surveyjs.io/form-library/examples/image-picker-question/) question with [`multiSelect`](https://surveyjs.io/form-library/documentation/api-reference/image-picker-question-model#multiSelect) enabled, respondents can choose multiple visual options. In some scenarios—such as logo recognition or brand association surveys—you may also want to include a 'None of these' option. This option allows respondents to indicate that none of the images match and stores a corresponding none item in the survey response.

However, unlike the standard [Checkbox](https://surveyjs.io/form-library/examples/create-checkboxes-question-in-javascript/) question type, the Image Picker does not provide a built-in [`showNoneItem`](https://surveyjs.io/form-library/documentation/api-reference/checkbox-question-model#showNoneItem) property. As a result, users can accidentally select both an image and a 'None' option, producing conflicting responses.

Requirements:
- Selecting 'None of these' deselects all other images.
- Selecting any image deselects 'None of these'.

## Solution

You can achieve this behavior by combining:
- An Image Picker question with [`multiSelect`](https://surveyjs.io/form-library/documentation/api-reference/image-picker-question-model#multiSelect) enabled.
- A standalone Checkbox question that contains only a custom 'None of these' option.
- Set up the [`resetValueIf`](https://surveyjs.io/form-library/documentation/api-reference/question#resetValueIf) property on both questions to make them mutually exclusive.

## Survey JSON Definition

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "panel",
          "name": "panel1",
          "questionTitleLocation": "hidden",
          "title": "Select all logos that apply to a statement",
          "elements": [
            {
              "type": "imagepicker",
              "name": "selectLogo",
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
              "resetValueIf": "{selectLogo} notempty",
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
  ]
}
```

## Live Demo

[View in Plunker](https://plnkr.co/edit/zonGQaaOV7SXSjTX)