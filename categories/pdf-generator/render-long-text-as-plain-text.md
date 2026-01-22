# Render Long Text Questions as Plain Text in PDF

## Problem

By default, a read-only [Long Text](https://surveyjs.io/form-library/examples/add-open-ended-question-to-a-form/) question is rendered in a PDF document with a visible border and background. This presentation is often unnecessary when the field is used purely for display purposes or to show long, read-only responses. I want to render the content as plain text while preserving natural line breaks.

## Solution

Register `FlatQuestionDefault` as a PDF renderer for the Long Text (comment) question type. This renderer outputs the question value as plain text without borders or background styling. Once registered, all Long Text questions in the generated PDF will be rendered as plain text with proper line wrapping and line breaks.

### Code Sample

```js
import { FlatRepository, FlatQuestionDefault } from "survey-pdf";

FlatRepository.register("comment", FlatQuestionDefault);
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/guapCoh7pcXDRpzd)