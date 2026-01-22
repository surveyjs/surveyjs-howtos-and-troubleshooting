# How to Render Comment Question as Plain Text in SurveyJS PDF Export

## Problem
By default, a read-only [Comment](https://surveyjs.io/form-library/examples/add-open-ended-question-to-a-form/) question (multi-line text) in SurveyPDF is rendered with a visible border and background — which often feels unnecessary when the field is used only for display or long read-only answers.

The goal is to render it as clean, simple plain text: no borders, no background, just the content with preserved natural line breaks.

## Solution

Register a custom flat renderer for the comment question type using `FlatQuestionDefault`.

### Code Sample

```js
import { FlatRepository, FlatQuestionDefault } from "survey-pdf";

FlatRepository.register("comment", FlatQuestionDefault);
```

### Survey JSON Schema

```json
{
  title: "Feedback Form",
  elements: [
    {
      type: "comment",
      name: "pdf_commentasplaintext",
      title: "Your detailed feedback",
      readOnly: true,
      defaultValue: "Sed venenatis nisl mi, eget lobortis augue venenatis ac.\n\nUt consectetur, nunc a tristique tempor, enim neque porttitor urna, non accumsan diam sem at erat. Suspendisse in sapien ac ligula aliquam porta a eu lorem"
    },
    {
      type: "comment",
      name: "suggestions",
      title: "Additional suggestions (plain text in PDF)",
      defaultValue: "Line one\nLine two\nLine three"
    }
  ]
}
```

### Live Demo

[View in Plunker](https://plnkr.co/edit/guapCoh7pcXDRpzd)