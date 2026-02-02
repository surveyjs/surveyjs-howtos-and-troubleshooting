# How to Check if a SurveyJS Element is a Page

## Problem

In SurveyJS, when working with survey elements programmatically, you may need to determine whether a given element is a page. This is common in custom renderers, question visibility logic, or survey manipulation. SurveyJS provides multiple ways to perform this check, but they are not all equivalent. Choosing the wrong method can lead to unexpected results.

---

## Solution

SurveyJS elements can be checked using the following approaches:

1. [`element.isPage`](https://surveyjs.io/form-library/documentation/api-reference/page-model#isPage) – A simple boolean property.  
2. [`element.getType() === "page"`](https://surveyjs.io/form-library/documentation/api-reference/question#getType) – Type string comparison.  
3. [`element.isDescendantOf("page")`](https://surveyjs.io/form-library/documentation/api-reference/question#isDescendantOf) – Type inheritance check.  
4. `element instanceof PageModel` – Standard JavaScript class check.  

### Code Sample

```ts
import { PageModel } from "survey-core";

// Suppose "element" is any SurveyJS element
function checkIfPage(element: any) {
  // 1. Using the shortcut property
  if (element.isPage) {
    console.log("Using isPage: This is a page!");
  }

  // 2. Using getType string comparison
  if (element.getType() === "page") {
    console.log("Using getType(): This is a page!");
  }

  // 3. Using isDescendantOf (type inheritance)
  if (element.isDescendantOf("page")) {
    console.log("Using isDescendantOf(): This is a page or inherits from it!");
  }

  // 4. Using instanceOf (JS class check)
  if (element instanceof PageModel) {
    console.log("Using instanceof PageModel: This is a page!");
  }
}

// ...
// Omitted: `SurveyModel` creation
// ...

survey.pages.forEach(page => {
  checkIfPage(page); 
  page.elements.forEach(el => checkIfPage(el)); // Child questions
});
```
### Survey JSON Schema
```
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        { "type": "text", "name": "q1", "title": "What is your name?" },
        { "type": "checkbox", "name": "q2", "title": "Select options", "choices": ["A", "B", "C"] }
      ]
    },
    {
      "name": "page2",
      "elements": [
        { "type": "radiogroup", "name": "q3", "title": "Pick one", "choices": ["Yes", "No"] }
      ]
    }
  ]
}
```

## Learn More

* [SurveyJS Form Library Documentation]()
* [SurveyJS PageModel API](https://surveyjs.io/form-library/documentation/api-reference/page-model)
* [JavaScript instanceof operator — MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof)