# Create an Offline Survey

## Problem

I need to distribute a survey for offline completion and store the collected responses locally.

## Solution

Use the [SurveyJS PDF Generator](https://surveyjs.io/pdf-generator/documentation/overview) to create a blank, fillable PDF document from your survey JSON schema.

```js
import { SurveyPDF } from "survey-pdf";

const surveyJson = { /* ... */ };
const surveyPdf = new SurveyPDF(surveyJson);
surveyPdf.save();
```

Once you have the PDF, choose one of the following distribution methods:

- **Option 1: Print the Form**\
Print the PDF and hand out physical copies to respondents. After completion, manually enter the responses into your system.

- **Option 2: Share the Digital Form**\
Send the fillable PDF to respondents who can complete it on a computer or mobile device. Ask them to save the completed form and return it via email or another file-sharing method. You can then either manually input the responses or extract them programmatically using libraries such as [`pdf-lib`](https://pdf-lib.js.org/) or [`PDF.js`](https://mozilla.github.io/pdf.js/).

## Learn More

- [SurveyJS PDF Generator Documentation](https://surveyjs.io/pdf-generator/documentation/overview)