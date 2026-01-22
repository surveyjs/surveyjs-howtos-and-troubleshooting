# Add Short Annotations Under PDF Form Fields in SurveyJS

## Problem

I want to display a short help text, hint, or annotation directly under a form field in a generated PDF document, rather than in the default location (typically below the question title).

## Solution

SurveyJS offers two approaches, depending on whether the annotation should appear in both the web survey and the PDF, or in the PDF only.

### Option 1: Define Annotations in the Survey JSON Schema

Choose this option if the annotation should be visible both in the web survey and in the generated PDF.

Add annotations to survey elements using the [`description`](https://surveyjs.io/form-library/documentation/api-reference/question#description) property. To position the description under the form field, set either the element-level [`descriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/question#descriptionLocation) or the survey-level [`questionDescriptionLocation`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#questionDescriptionLocation) property to `"underInput"`. The survey-level setting applies to all question descriptions unless overridden at the question level.

```json
{
  "title": "User Feedback Survey",
  "questions": [
    {
      "type": "text",
      "name": "fullName",
      "title": "Full name",
      "description": "Please enter your first and last name.",
      "descriptionLocation": "underInput"
    },
    {
      "type": "radiogroup",
      "name": "satisfaction",
      "title": "How satisfied are you with our service?",
      "description": "Select the option that best matches your experience.",
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
      "name": "additionalFeedback",
      "title": "Additional feedback",
      "description": "Share any comments or suggestions you may have.",
      "descriptionLocation": "underInput"
    }
  ],
  "questionDescriptionLocation": "underInput"
}
```

[Open in Plunker](https://plnkr.co/edit/wJjTt4kn7Sc8bNum)

### Option 2: Add Annotations Only in the Generated PDF

Choose this option if the annotation should appear only in the PDF, without affecting the web survey UI.

To do this, customize PDF rendering by handling the [`SurveyPDF.onRenderQuestion`](https://surveyjs.io/pdf-generator/documentation/api-reference/surveypdf#onRenderQuestion)) event. This event is raised once for each question during PDF generation.

The event handler receives an `options` object with the following properties as the second parameter:

- `options.question`        
A survey question that is being rendered.

- `options.bricks`          
An array of [PDF bricks](https://surveyjs.io/pdf-generator/documentation/api-reference/pdfbrick) used to render the question. Bricks are low-level layout elements with specified content, size, and location. You can modify or append bricks to customize rendering.

- `options.point`       
Coordinates of the top-left corner of the rendered question: `{ xLeft: number, yTop: number }`.

- `options.controller`      
A [`DocController`](https://surveyjs.io/pdf-generator/documentation/api-reference/doccontroller) object that provides access to main PDF document properties (font, margins, page width and height) and allows you to modify them.

- `options.repository`          
A factory for creating PDF rendering components. Use its `create` method to generate custom bricks.

```js
import { Model } from "survey-core";
import { SurveyPDF, SurveyHelper } from "survey-pdf";

function saveSurveyToPdf (filename, surveyModel) {
    const pdfDoc = new SurveyPDF(json);
    if (surveyModel) {
      pdfDoc.data = surveyModel.data;
      pdfDoc.locale = surveyModel.locale;
    }

    // Add an annotation under a specific question
    pdfDoc.onRenderQuestion.add((_, options) => {
      if (options.question.name !== "pdf_bottomdesc") return;

      const plainBricks = options.bricks[0].unfold();
      const lastBrick = plainBricks[plainBricks.length - 1];
      const point = SurveyHelper.createPoint(lastBrick);

      return new Promise(resolve => {
        SurveyHelper
          .createDescFlat(
            point,
            options.question,
            options.controller,
            'Short annotation under the form field'
          )
          .then(descBrick => {
            options.bricks.push(descBrick);
            resolve();
          });
      });
    });
    
    pdfDoc.save(filename);
}

const surveyJson = {
  "elements": [
    {
      "type": "text",
      "title": "Add a short annotation under the form field",
      "name": "pdf_bottomdesc"
    }
  ]
};

const survey = new Model(surveyJson);
survey.addNavigationItem({
    id: "download-pdf",
    title: "Download PDF",
    action: () => {
        saveSurveyToPdf("surveyResult.pdf", survey);
    }
});
```

[Open in Plunker](https://plnkr.co/edit/rVqnH3erg24p1aaW)