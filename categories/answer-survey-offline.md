# Export and Complete Survey as Fillable PDF

## Problem
How to create a survey, distribute it to respondents for offline completion, and save responses locally?

## Solution
To enable offline survey completion, export the survey as a fillable PDF using the [SurveyJS PDF Generator](https://surveyjs.io/pdf-generator/examples/save-completed-forms-as-pdf-files/reactjs). Generate a blank PDF by initializing the [`SurveyPDF`](https://surveyjs.io/pdf-generator/documentation/api-reference/surveypdf) object without setting the `surveyPDF.data` property. Distribute the PDF to respondents, who can fill it offline using any PDF reader and return it via email. Upon receiving the filled PDF, manually input the responses into your system or programmatically parse the PDF using tools like pdf-lib or PDF.js.

## Code Sample
```javascript
// Export a blank survey as a fillable PDF
const surveyPDF = new SurveyPDF.SurveyPDF(surveyJson);
// Do not set surveyPDF.data for a blank form
surveyPDF.save();
```