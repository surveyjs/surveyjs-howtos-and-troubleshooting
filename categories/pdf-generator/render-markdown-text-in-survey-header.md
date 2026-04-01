# Render Markdown in PDF

## Problem

I need to display richly formatted text (instructions, descriptions, or comments) defined in Markdown within a SurveyJS form. This content may include dynamic values (for example, user name or current date) and must render correctly both in the web UI and in generated PDF documents.

## Solution

Use SurveyJS for form configuration and convert Markdown to HTML at runtime using a third-party library. This approach enables Markdown syntax (bold, italic, etc.) and supports dynamic expressions while ensuring consistent rendering in both environments.

Implementation steps:

1. Define your survey JSON and include Markdown content in titles or descriptions.
2. Use expression questions to provide dynamic values (for example, current date).
3. Convert Markdown to HTML during rendering using a third-party library (for example,[markdown-it](https://github.com/markdown-it/markdown-it)).
4. Sanitize the generated HTML before assigning it to the output.

### Code Sample

```javascript
import markdownit from "markdown-it";
import { SurveyPDF } from "survey-pdf";

const surveyJson = { /* Survey JSON schema */ };
const pdfOptions = { /* PDF configuration */ };

const converter = markdownit({
  html: true, // Allow HTML in Markdown source
});

const surveyPDF = new SurveyPDF(surveyJson, pdfOptions);

// Provide values for dynamic placeholders
surveyPDF.setValue("id", "123");
surveyPDF.setValue("userName", "User Name");

surveyPDF.onTextMarkdown.add((_, options) => {
  // Convert Markdown to HTML
  let str = converter.renderInline(options.text);
  // ...
  // Omitted: HTML sanitization
  // ...
  options.html = str;
});
```

### Survey JSON Schema

```json
{
  "title": "User Feedback Survey",
  "description": "Name: **{userName}**. Today's date: **{todaysDate}**. ID: **{id}**",
  "pages": [
    {
      "name": "user_info",
      "elements": [
        {
          "type": "expression",
          "name": "todaysDate",
          "visible": false,
          "expression": "currentDate()",
          "displayStyle": "date"
        },
        {
          "type": "text",
          "name": "name",
          "title": "Name",
          "description": "Enter your full name.",
          "isRequired": true
        },
        {
          "type": "text",
          "name": "label",
          "title": "Label",
          "description": "Enter the relevant label for your role or department."
        },
        {
          "type": "text",
          "name": "date",
          "title": "Date",
          "description": "Select today's date.",
          "inputType": "date"
        },
        {
          "type": "text",
          "name": "id",
          "title": "ID",
          "description": "Enter your unique ID."
        }
      ]
    },
    {
      "name": "feedback",
      "elements": [
        {
          "type": "comment",
          "name": "comments",
          "title": "Please provide your feedback",
          "description": "Share your thoughts about our product or service. Markdown is supported here as well."
        },
        {
          "type": "rating",
          "name": "satisfaction",
          "title": "Overall satisfaction",
          "minRateDescription": "Very dissatisfied",
          "maxRateDescription": "Very satisfied"
        }
      ]
    }
  ]
}
```

## Live Demo

[Open in CodeSandbox](https://codesandbox.io/p/sandbox/pensive-noether-gdk8sh?file=%2Fsrc%2Findex.js%3A28%2C3-39%2C6)

## Learn More

- [Demo: Convert Markdown to HTML with markdown-it](https://surveyjs.io/form-library/examples/edit-survey-questions-markdown/)
- [Demo: Convert Markdown to HTML with Marked](https://surveyjs.io/form-library/examples/enable-markdown-in-surveys/)