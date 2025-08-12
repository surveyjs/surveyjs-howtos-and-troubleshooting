# Dropdown Popup Appears Incorrectly Unlinked from the Input Field When the Parent Container Is Scrolling

## Problem
A dropdown or multi-select dropdown question is added to a survey. If you activate the popup and start scrolling the parent container, the popup doesn't change its location following the input field and remains at its original position.

## Solution
This occurs because the survey, embedded within a scrollable parent container, cannot detect scroll events, preventing the popup from closing or repositioning correctly.
To address this issue, implement the [`SurveyModel.onPopupVisibleChanged`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onPopupVisibleChanged) event to obtain a popup model. Subscribe to the parent container's scroll event and hide the popup model when a parent container is scrolled.

## Code Sample
```javascript
import { SurveyModel } from "survey-core";

// Survey JSON with dropdown
const surveyJson = {
  pages: [
    {
      name: "page1",
      elements: [
        {
          type: "dropdown",
          name: "question1",
          choices: [
            // Omitted for brevity
          ]
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
    if (hidePopup) {
      hidePopup();
    }
  };

  survey.onPopupVisibleChanged.add((_, opt) => {
    if (opt.visible) {
      hidePopup = () => opt.popup.hide();
      container.addEventListener("scroll", handleScroll);
    } else {
      hidePopup = null;
      container.removeEventListener("scroll", handleScroll);
    }
  });

  // Cleanup event listener when survey is disposed
  survey.onDispose.add(() => {
    container.removeEventListener("scroll", handleScroll);
  });
}

const survey = new SurveyModel(surveyJson);
  handleDropdownPopupOnScroll(survey, ".scrollable-container"); // Pass your container selector
} catch (error) {
  console.error("Failed to initialize survey:", error);
}
```
