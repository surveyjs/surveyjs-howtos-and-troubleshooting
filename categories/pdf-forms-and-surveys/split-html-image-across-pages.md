# Split Image Across Multiple PDF Pages

## Problem

When exporting a SurveyJS survey to PDF using the [SurveyJS PDF Generator](https://surveyjs.io/pdf-generator/documentation/overview) library, large images in `html` type questions often do not fit on a single page.  
By default, SurveyPDF does not automatically split such images — they may get cut off or cause layout issues.

**Important note**: This splitting functionality is only supported for questions of type [`"html"`](https://surveyjs.io/form-library/documentation/api-reference/add-custom-html-to-survey).  
If you're using the [`"image"`](https://surveyjs.io/form-library/documentation/api-reference/add-image-to-survey) question type, convert it to an `html` question with an `<img>` tag.

## Solution

To split an image, implement the [`onRenderQuestion`](https://surveyjs.io/pdf-generator/documentation/api-reference/surveypdf#onRenderQuestion) function of SurveyPDF.  

The general approach:
1. Detect HTML questions containing large images
2. Load the original image
3. Calculate how much vertical space remains on the current page
4. Crop the image into multiple parts (one per page)
5. Create new `HTMLBrick` instances with base64 image chunks
6. Replace original brick with multiple page-split bricks


### Code Sample

```typescript
function createSurveyPdfModel(surveyModel) {
    const pdfWidth = surveyModel?.pdfWidth ?? 210;
    const pdfHeight = surveyModel?.pdfHeight ?? 297;

    const options = {
        fontSize: 14,
        margins: { left: 10, right: 10, top: 10, bot: 10 },
        format: [pdfWidth, pdfHeight],
        htmlRenderAs: 'image',  
    };

    const surveyPDF = new SurveyPDF.SurveyPDF(json, options);

    if (surveyModel) {
        surveyPDF.data = surveyModel.data;
        surveyPDF.locale = surveyModel.locale;
    }

    // Helper: Draw portion of image to canvas and create base64 chunk
    function drawPart(
        brickHeight,
        multiplier,
        shift,
        canvas,
        img,
        firstBrickHeight,
        fullPagesCnt,
        onePageSize,
        htmlBrick,
        controller
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

        return new SurveyPDF.HTMLBrick(null, controller, brickRect, brickHtml, true);
    }

    // Helper: Load image from base64
    async function getImage(imgBase64) {
        const img = new Image();
        img.src = imgBase64;
        await new Promise((resolve, reject) => {
            img.onload = () => resolve(img);
            img.onerror = reject;
        });
        return img;
    }

    surveyPDF.onRenderQuestion.add(async (survey, options) => {
        if (options.question.getType() !== 'html') return;

        const imageBrick = options.bricks[0];
        if (!imageBrick || !imageBrick.image) return;

        options.bricks = []; // Clear original brick

        const img = await getImage(imageBrick.image);

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

        const onePageSize = fullPageHeight / multiplier;

        // Height that fits on first (current) page
        const firstBrickHeight = Math.min(img.height, (fullPageHeight - currY) / multiplier);

        // First chunk
        const firstBrick = drawPart(
            firstBrickHeight, multiplier, 0, canvas, img,
            0, 0, onePageSize, imageBrick, options.controller
        );
        options.bricks.push(firstBrick);

        // Remaining chunks
        let cnt = 0;
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

    return surveyPDF;
}

function saveSurveyToPdf(filename, surveyModel) {
    createSurveyPdfModel(surveyModel).save(filename);
}

// Optional: Improve image quality (increases file size)
SurveyPDF.SurveyHelper.HTML_TO_IMAGE_QUALITY = 3;
```

### Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "html",
          "name": "question1",
          "html": "<p>The image below doesn't fit onto one PDF page.</p><img  width='579px' height='1416px' src='data:image/jpeg;base64,...'/>"
        }
      ]
    }
  ]
}
```

### Live Demo

[View in Plunker](https://plnkr.co/edit/8W9OGmCembH1yvWD)