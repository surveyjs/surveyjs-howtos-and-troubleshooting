# Add Short Annotations Under PDF Form Fields in SurveyJS

## Problem

I want to display a short help text, hint, or annotation directly under a form field in a generated PDF document, rather than in the default location (typically below the question title).

## Solution

Add annotations to survey elements using the [`description`](https://surveyjs.io/form-library/documentation/api-reference/question#description) property. To position the description under the form field, set either the element-level [`descriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/question#descriptionLocation) or the survey-level [`questionDescriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#questionDescriptionLocation) property to `"underInput"`. The survey-level setting applies to all question descriptions unless overridden at the question level.

This placement is respected both in the web survey and in the generated PDF.

### Survey JSON Schema

```json
{
  "title": "User Feedback Survey",
  "questionDescriptionLocation": "underInput",
  "elements": [
    {
      "type": "text",
      "name": "fullName",
      "title": "Full name",
      "description": "Please enter your first and last name."
    },
    {
      "type": "radiogroup",
      "name": "satisfaction",
      "title": "How satisfied are you with our service?",
      "description": "Select the option that best matches your experience.",
      "choices": [
        "Very satisfied",
        "Satisfied",
        "Neutral",
        "Dissatisfied",
        "Very dissatisfied"
      ]
    },
    {
      "type": "comment",
      "name": "additionalFeedback",
      "title": "Additional feedback",
      "description": "Share any comments or suggestions you may have."
    }
  ]
}
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/wJjTt4kn7Sc8bNum)

## Learn More

- [`description`](https://surveyjs.io/form-library/documentation/api-reference/question#description)
- [`descriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/question#descriptionLocation)
- [`questionDescriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#questionDescriptionLocation)
