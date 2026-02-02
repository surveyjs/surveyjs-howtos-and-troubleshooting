# How to Check a SurveyJS Element Type

## Problem

In SurveyJS, when working with survey elements programmatically, you may need to determine whether a given element is an instance of a specific question type, or a page SurveyJS provides multiple ways to perform this check.

## Solution

To check whether an element is a question, panel, page, or survey, or to check an element's type, use the following APIs:

1. `is~` properties of a survey element.
* [`SurveyElement.isQuestion`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isQuestion)
* [`SurveyElement.isPanel`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isPanel)
* [`SurveyElement.isPage`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isPage)
* [`SurveyElement.isSurvey`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isSurvey)


2. [`SurveyElement.getType(type)`](https://surveyjs.io/form-library/documentation/api-reference/question#getType)
Returns the type of the current element as a string, e.g., `"page"`, `"panel"`, `"text"`, etc. All types are listed in the [`getType()` documentation](https://surveyjs.io/form-library/documentation/api-reference/question#getType).

```js
element.getType()
// Returns "page" for a survey page
```

3. [`SurveyElement.isDescendantOf(type)`](https://surveyjs.io/form-library/documentation/api-reference/question#isDescendantOf)

Checks if the type of the element is "type" or inherits from it. For example, if an element's type is "page", then both `isDescendantOf("page")` and `isDescendantOf("panel")` return `true` because [`PageModel`](https://surveyjs.io/form-library/documentation/api-reference/page-model) extends [`PanelModel`](https://surveyjs.io/form-library/documentation/api-reference/panel-model).

```js
const page = survey.getPageByName("page1");

page.isDescendantOf("page");  // true, because getType() returns "page"
page.isDescendantOf("panel"); // true, because PageModel extends PanelModel
```

4. `instanceof`

Standard JavaScript class check. Requires that a target object model is imported and available in your code.

The following returns true if element is a PageModel or a subclass of it:
```js
import { PageModel } from "survey-core";

const isPage = element instanceof PageModel;
```

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

* [SurveyJS Form Library Documentation](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model)
* [SurveyJS PageModel API](https://surveyjs.io/form-library/documentation/api-reference/page-model)
* [JavaScript instanceof operator — MDN Reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof)