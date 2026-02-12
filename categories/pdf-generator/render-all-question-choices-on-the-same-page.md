# How to Keep All Survey Choices on a Single Page in SurveyPDF

## Problem

When creating a survey with multiple-choice questions, such as radiogroup or checkbox questions, SurveyPDF may automatically split the options across multiple pages if there are many of them or if the layout space is limited.

This can cause issues when:

* Printing the survey on paper for respondents.
* Wanting to maintain a concise, easy-to-read layout.

The goal is to keep all choices of a question on the same page, no matter how many options exist.

## Solution

SurveyPDF provides an event called [`onRenderQuestion`](https://surveyjs.io/pdf-generator/documentation/api-reference/surveypdf#onRenderQuestion) that allows you to customize how each question is rendered. By using this event, you can force all option bricks of a question to stay together on the same page.

Implement the `onRenderQuestion` function and create a `CompositeBrick` for the question, which groups all its option bricks as a single block that won't break across pages.

### Code Sample
```javascript
import { SurveyPDF, CompositeBrick } from "survey-pdf";

const pdfDocOptions = { ... };
const surveyPdf = new SurveyPDF(surveyJson, pdfDocOptions);

surveyPDF.onRenderQuestion.add((_, options) => {
    // Check if the question is of type radiogroup or checkbox
    if (options.question.isDescendantOf("radiogroup") || options.question.isDescendantOf("checkbox")) {
        // Combine all option bricks into a single composite brick
        options.bricks = [new CompositeBrick(...options.bricks)];
    }
});
```

### Live Demo

[View in Plunker](https://plnkr.co/edit/9dwWgrJrG550EiX4)
