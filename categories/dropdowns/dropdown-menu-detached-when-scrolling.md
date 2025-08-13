# Dropdown Menu Appears Detached from Input Field When Parent Container Scrolls

## Problem

I added a dropdown or multi-select dropdown question to a survey. When I open the menu and then scroll the parent container, the menu stays in its original position instead of moving with the input field or closing.

## Solution

This happens because the survey is embedded in a scrollable parent container that it cannot detect scroll events from. As a result, the menu isn't closed or repositioned when the container scrolls.

To fix this, handle the `SurveyModel`'s [`onPopupVisibleChanged`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onPopupVisibleChanged) event to get the popup model. Then, subscribe to the parent container's scroll event and hide the menu whenever scrolling occurs.

### Code Sample

```javascript
import { Model } from "survey-core";

const surveyJson = {
  pages: [
    {
      name: "page1",
      elements: [
        {
          type: "dropdown",
          name: "question1",
          choices: [ /* An array of choice options */ ]
        }
      ]
    }
  ]  
};

function handleDropdownPopupOnScroll(survey, containerSelector) {
  const container = document.querySelector(containerSelector);
  if (!container) {
    console.warn(`Container with selector "${containerSelector}" not found.`);
    return;
  }

  let hidePopup = null;

  const handleScroll = () => {
    if (hidePopup) hidePopup();
  };

  survey.onPopupVisibleChanged.add((_, options) => {
    if (options.visible) {
      hidePopup = () => options.popup.hide();
      container.addEventListener("scroll", handleScroll);
    } else {
      hidePopup = null;
      container.removeEventListener("scroll", handleScroll);
    }
  });

  // Clean up when survey is disposed
  survey.onDispose.add(() => {
    container.removeEventListener("scroll", handleScroll);
  });
}

try {
  const survey = new Model(surveyJson);
  handleDropdownPopupOnScroll(survey, ".scrollable-container"); // Pass your container selector
} catch (error) {
  console.error("Failed to initialize survey:", error);
}
```
