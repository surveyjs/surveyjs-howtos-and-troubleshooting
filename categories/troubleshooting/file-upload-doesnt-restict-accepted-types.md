# Restrict File Uploads to Specific Types in SurveyJS

## Problem
When configuring the [`acceptedTypes`](https://surveyjs.io/form-library/documentation/api-reference/file-model#acceptedTypes) property for a [File Upload](https://surveyjs.io/form-library/documentation/api-reference/file-model) question in SurveyJS to allow only specific file types (e.g., `.pdf`), users can still bypass the filter in the file chooser dialog by selecting "All Files" and uploading incorrect file types.

## Solution
The `acceptedTypes` property sets the `accept` attribute for the `<input>` element, which filters the file chooser dialog but does not enable validation. To restrict uploads to specific file types, implement the [`survey.onUploadFiles`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUploadFiles) function and validate the file type before uploading. If a user attempts to upload invalid files, send `options.callback` with a custom error message.

### Code Sample
```javascript
survey.onUploadFiles.add((_, options) => {
  let formData = new FormData();
  let file = options.files[0];
  if (!file || !file.name || file.name.toLowerCase().endsWith(".pdf")) {
    formData.append(file.name, file);
  } else {
    options.callback([], ["Only PDF files are allowed"]);
    return;
  }
  let xhr = new XMLHttpRequest();
  xhr.open("POST", "https://api.surveyjs.io/private/Surveys/uploadTempFiles");
  xhr.onload = function () {
    let data = JSON.parse(xhr.responseText);
    options.callback(
      "success",
      options.files.map(function (file) {
        return { file: file, content: data[file.name] };
      })
    );
  };
  xhr.send(formData);
});
```

### Survey JSON Schema
```json
{
  "elements": [
    {
      "type": "file",
      "title": "Upload your document",
      "storeDataAsText": false,
      "allowMultiple": false,
      "maxSize": 102400,
      "acceptedTypes": ".pdf"
    }
  ]
}
```