# Save Photo Capture Timestamp in SurveyJS File Upload

## Problem

My survey includes a File Upload question with the built-in [photo capture functionality](https://surveyjs.io/form-library/examples/photo-capture/). In addition to storing the uploaded image, I need to persist the timestamp indicating when the user captured the photo.

## Solution

To store the photo capture timestamp, handle the `SurveyModel`'s [`onUploadFiles`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUploadFiles) event. In the event handler, upload the files to your storage backend and add the timestamp recorded at upload time to the survey results.

Instead of saving a plain file URL in the survey results, this approach stores an object that contains both the file URL and the timestamp. However, this change affects file previews because SurveyJS expects the file value to be either a URL or a base64-encoded string. To restore preview functionality, also handle the [`onDownloadFile`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onDownloadFile) event. In this handler, extract the file URL from the stored object and explicitly fetch the image to display it in the preview.

### Code Sample

```javascript
// ...
// Omitted: `Survey.Model` creation
// ...
const ENDPOINT_URL = "https://api.surveyjs.io/private/Surveys/";

survey.onUploadFiles.add((_, options) => {
  const formData = new FormData();

  options.files.forEach((file) => {
    formData.append(file.name, file);
  });

  fetch(ENDPOINT_URL + "uploadTempFiles", {
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
            url: ENDPOINT_URL + "/getTempFile?name=" + data[file.name],
            uploadedAt
          }
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

```json
{
  "elements": [
    {
      "type": "file",
      "title": "Upload your photo",
      "name": "photo",
      "storeDataAsText": false,
      "waitForUpload": true,
      "sourceType": "camera" // or "file-camera"
    }
  ]
}
```

## Live Demo

[Open in CodePen](https://codepen.io/JaneGaid/pen/pvbRmBo)

## Learn More

- [Demo: Retrieve Uploaded Files for Preview Using File Names](https://surveyjs.io/form-library/examples/store-file-names-in-survey-results/)
- [`QuestionFileModel` API Documentation](https://surveyjs.io/form-library/documentation/api-reference/file-model)