# Capture Photo Capture Timestamp in SurveyJS

## Problem

I have a photo capture `"file"` question. How to save the actual timestamp when the user captures the photo?

## Solution

To persist the timestamp for the moment when a user captures a photo, implement the SurveyModel’s [`onUploadFiles`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUploadFiles) event. Within this event, uploads files to a storage and store the current timestamp and the resulting file URLs in the survey results.
To display a file preview, implement the [`onDownloadFile](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onDownloadFile) function, unwrap the new content structure, extract the URL and download an image for preview.

### Code Sample

```javascript
survey.onUploadFiles.add((_, options) => {
  const formData = new FormData();

  options.files.forEach((file) => {
    formData.append(file.name, file);
  });

  fetch("https://api.surveyjs.io/private/Surveys/uploadTempFiles", {
    method: "POST",
    body: formData,
  })
    .then((response) => response.json())
    .then((data) => {
      const uploadedAt = new Date().toISOString();

      options.callback(
        options.files.map((file) => ({
          file,
          content: {
            url:
              "https://api.surveyjs.io/private/Surveys/getTempFile?name=" +
              data[file.name],
            uploadedAt,
          },
        }))
      );
    })
    .catch((error) => {
      console.error("Error: ", error);
      options.callback([], ["An error occurred during photo upload."]);
    });
});

survey.onDownloadFile.add((_, options) => {
  const fileUrl =
    typeof options.content === "string"
      ? options.content
      : options.content?.url;

  if (!fileUrl) {
    options.callback("error");
    return;
  }

  fetch(fileUrl)
    .then((response) => response.blob())
    .then((blob) => {
      const file = new File([blob], options.fileValue.name, {
        type: options.fileValue.type,
      });

      const reader = new FileReader();
      reader.onload = (e) => {
        options.callback("success", e.target?.result);
      };
      reader.readAsDataURL(file);
    })
    .catch((error) => {
      console.error("Error: ", error);
      options.callback("error");
    });
});
```
### Survey JSON Schema
```
{
  "elements": [
    {
      "type": "file",
      "title": "Upload your photo",
      "name": "photo",
      "storeDataAsText": false,
      "waitForUpload": true,
      "sourceType": "camera"
    }
  ]
}
```
## Live Demo

[View in Plunker](https://codepen.io/JaneGaid/pen/pvbRmBo)

## Learn More
* [Demo: Photo Capture](https://surveyjs.io/form-library/examples/photo-capture/)
* [Demo: Retrieve Uploaded Files for Preview Using File Names](https://surveyjs.io/form-library/examples/store-file-names-in-survey-results/)
* [API Documentation: QuestionFileModel](https://surveyjs.io/form-library/documentation/api-reference/file-model)