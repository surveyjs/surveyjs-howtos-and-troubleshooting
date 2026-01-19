# Add a Short Annotation / Help Text Under a Form Field in SurveyJS

## Problem
How to display a short help text, hint, or annotation directly under a form field (below the input area), instead of the default location (usually below the title).

Common use cases:

- Explanatory notes
- Additional instructions without cluttering the question title

## Solution
To achieve your goal, define the following properties within a question configuration object:

* [`description`](https://surveyjs.io/form-library/documentation/api-reference/question#description) — the actual text of the annotation/help text.
* [`descriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/question#descriptionLocation): "underInput" — places the description under the input/control instead of under the title.

This combination works for any question types: text, comment, radiogroup, checkbox, dropdown, rating, etc.

## Code Sample

```
function createSurveyPdfModel(surveyModel) {       
    const options = {
        fontSize: 14,
        margins: {
            left: 10,
            right: 10,
            top: 10,
            bot: 10
        },
        format: [210, 297],        
    };
    const surveyPDF = new SurveyPDF.SurveyPDF(json, options);
    if (surveyModel) {
        surveyPDF.data = surveyModel.data;
        surveyPDF.locale = surveyModel.locale;
    }    
    return surveyPDF;
}

function saveSurveyToPdf(filename, surveyModel) {
    createSurveyPdfModel(surveyModel).save(filename);
}
```

### Survey JSON Schema

```
{
  "title": "Customer Satisfaction Survey",
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "text",
          "name": "fullName",
          "title": "Full name",
          "description": "Please enter your first and last name (max 100 characters)",
          "descriptionLocation": "underInput",
          "maxLength": 100
        },
        {
          "type": "radiogroup",
          "name": "satisfaction",
          "title": "How satisfied are you with our service?",
          "description": "Choose the option that best reflects your recent experience",
          "descriptionLocation": "underInput",
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
          "name": "suggestions",
          "title": "What could we improve?",
          "description": "Your detailed feedback helps us get better — thank you!",
          "descriptionLocation": "underInput",
          "rows": 5
        }
      ]
    }
  ]
}
```
### Live demo

[View in Plunker](https://plnkr.co/edit/aKGhfKyNUhxXm9g7)