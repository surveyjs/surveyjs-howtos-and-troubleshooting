# Prevent Choice Options from Splitting Across PDF Pages in SurveyJS PDF Generator

## Problem

When generating a PDF that contains Radio Button Group or Checkboxes questions, SurveyJS PDF Generator may split a question's choice options across pages if a page break occurs during rendering. I need to ensure that all choice options for a given question remain on the same page, even if this causes the entire question to move to the next page.

## Solution

SurveyJS PDF Generator exposes the [`onRenderQuestion`](https://surveyjs.io/pdf-generator/documentation/api-reference/surveypdf#onRenderQuestion) event, which allows you to override the default rendering logic for individual questions. Within this event handler, you can wrap all generated option bricks in a `CompositeBrick`. A `CompositeBrick` groups multiple bricks into a single rendering unit that cannot be split across pages, ensuring the question's choices are kept together.

### Code Sample

```javascript
import { SurveyPDF, CompositeBrick } from "survey-pdf";

const pdfDocOptions = { /* ... */ };
const pdfDoc = new SurveyPDF(surveyJson, pdfDocOptions);

pdfDoc.onRenderQuestion.add((_, options) => {
  if (
    options.question.isDescendantOf("radiogroup") ||
    options.question.isDescendantOf("checkbox")
  ) {
    options.bricks = [new CompositeBrick(...options.bricks)];
  }
});
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/OiE5WX51PUqdeRSh)
