# Split a Long Image Across Multiple PDF Pages

## Problem

When exporting a survey to PDF, large images embedded via the HTML survey element may not fit on a single page. In this case, the image is clipped at the page boundary instead of continuing on the next page. I want to automatically split such images, so they flow across multiple PDF pages.

## Solution

> Image splitting is supported only for survey elements of the [HTML type](https://surveyjs.io/form-library/documentation/api-reference/add-custom-html-to-survey). If you use the [Image](https://surveyjs.io/form-library/documentation/api-reference/add-image-to-survey) element, convert it to an HTML element and include the image using an `<img>` tag.

To split an image across pages, handle the [`onRenderQuestion`](https://surveyjs.io/pdf-generator/documentation/api-reference/surveypdf#onRenderQuestion) event of `SurveyPDF` and replace the original image brick with multiple page-sized bricks.

To implement this behavior, follow these steps:

1. Detect HTML questions that contain an image.
2. Load the image into memory.
3. Determine the remaining vertical space on the current PDF page.
4. Split the image into multiple vertical segments.
5. Convert each segment to a Base64 image.
6. Create multiple `HTMLBrick` instances&mdash;one per page segment.
7. Replace the original brick with the generated page-split bricks.

### Code Sample

```js
import { Model } from "survey-core";
import { SurveyPDF, HTMLBrick, SurveyHelper } from "survey-pdf";

function saveSurveyToPdf(filename, surveyModel) {
  const options = {
    htmlRenderAs: "image",  
  };

  const pdfDoc = new SurveyPDF(json, options);

  if (surveyModel) {
      pdfDoc.data = surveyModel.data;
      pdfDoc.locale = surveyModel.locale;
  }

  // Draw a portion of the image
  function drawPart(
    brickHeight, multiplier,
    shift, canvas,
    img, firstBrickHeight,
    fullPagesCnt, onePageSize,
    htmlBrick, controller
  ) {
    canvas.height = brickHeight;
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(
      img,
      0, shift, img.width, brickHeight,
      0, 0, img.width, brickHeight
    );

    const currImageBase64 = canvas.toDataURL('image/jpeg');

    const brickHtml = `
      <img src='${currImageBase64}' 
        width='${img.width * multiplier}' 
        height='${brickHeight * multiplier}' />`;

    const brickRect = {
      xLeft: htmlBrick.xLeft,
      xRight: htmlBrick.xRight,
      yTop: htmlBrick.yTop + multiplier * (firstBrickHeight + fullPagesCnt * onePageSize),
      yBot: htmlBrick.yTop + multiplier * (firstBrickHeight + fullPagesCnt * onePageSize + brickHeight),
    };

    return new HTMLBrick(null, controller, brickRect, brickHtml, true);
  }

  // Load the image in Base64 format 
  async function getImage(imgBase64) {
    const img = new Image();
    img.src = imgBase64;
    await new Promise((resolve, reject) => {
      img.onload = () => resolve(img);
      img.onerror = reject;
    });
    return img;
  }

  pdfDoc.onRenderQuestion.add(async (_, options) => {
    // Split only HTML elements
    if (options.question.getType() !== 'html') return;

    // Get the HTML brick
    const imageBrick = options.bricks[0];
    if (!imageBrick || !imageBrick.image) return;

    options.bricks = []; // Clear original brick

    // Extract image from the HTML brick
    const img = await getImage(imageBrick.image);

     // Calculate the available page height and the coordinate at which to start drawing
    const fullPageHeight =
      options.controller.paperHeight -
      options.controller.margins.top -
      options.controller.margins.bot;
    const currY = imageBrick.yTop - fullPageHeight * Math.floor(imageBrick.yTop / fullPageHeight);

    const canvas = document.createElement('canvas');
    canvas.width = img.width;

    // Scale down if image is wider than page
    let multiplier = 1;
    if (imageBrick.width < img.width) {
      multiplier = imageBrick.width / img.width;
    }

    // Calculate what portion of the original image would fit onto one page
    const onePageSize = fullPageHeight / multiplier;

    // Calculate the height of the first brick
    const firstBrickHeight = Math.min(img.height, (fullPageHeight - currY) / multiplier);

    // Draw the first brick
    const firstBrick = drawPart(
      firstBrickHeight, multiplier, 0, canvas, img,
      0, 0, onePageSize, imageBrick, options.controller
    );
    options.bricks.push(firstBrick);

    // Page counter
    let cnt = 0;
    // Draw the rest of the bricks
    for (let i = firstBrickHeight; i < img.height; i += onePageSize) {
      const brickHeight = Math.min(img.height - i, onePageSize);

      const currBrick = drawPart(
        brickHeight, multiplier, i, canvas, img,
        firstBrickHeight, cnt, onePageSize, imageBrick, options.controller
      );

      options.bricks.push(currBrick);
      cnt += 1;
    }
  });

  pdfDoc.save(filename);
}

// Optional: Improve image quality (increases file size)
SurveyHelper.HTML_TO_IMAGE_QUALITY = 3;

const surveyJson = {
  "elements": [
    {
      "type": "html",
      "name": "question1",
      "html": "<p>The image below doesn't fit onto one PDF page.</p><img width='579px' height='1416px' src='data:image/jpeg;base64,...'/>"
    }
  ]
};

const survey = new Model(surveyJson);
survey.addNavigationItem({
  id: "download-pdf",
  title: "Download PDF",
  action: () => {
    saveSurveyToPdf("surveyResult.pdf", survey);
  }
});
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/pg45sNrlLa0Yv3fo)