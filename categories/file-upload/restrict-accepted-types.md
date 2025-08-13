# Restrict File Uploads to Specific Types in SurveyJS

## Problem

When you configure the [`acceptedTypes`](https://surveyjs.io/form-library/documentation/api-reference/file-model#acceptedTypes) property for a File Upload question in SurveyJS (e.g., to allow only `.pdf` files), the file chooser dialog filters visible files accordingly.
However, users can still bypass this filter by selecting "All Files" and uploading unsupported file types.

## Solution

The `acceptedTypes` property only sets the `accept` attribute on the underlying `<input>` element, which filters file options in the dialog but does not enforce validation. To strictly allow only certain file types, handle the `SurveyModel`'s [`onUploadFiles`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUploadFiles) event and check each file's extension or MIME type before uploading. If a file is invalid, call `options.callback` with an empty array and a custom error message.

The example below accepts only PDF files and rejects all others.

### Code Sample

```javascript
// ...
// Omitted: `Survey.Model` creation
// ...

survey.onUploadFiles.add((_, options) => {
  const formData = new FormData();
  cinst file = options.files[0];
  if (!file || !file.name || file.name.toLowerCase().endsWith(".pdf")) {
    formData.append(file.name, file);
  } else {
    options.callback([], ["Only PDF files are allowed"]);
    return;
  }
  const xhr = new XMLHttpRequest();
  xhr.open("POST", "https://api.surveyjs.io/private/Surveys/uploadTempFiles");
  xhr.onload = function () {
    const data = JSON.parse(xhr.responseText);
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