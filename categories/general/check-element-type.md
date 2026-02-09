# How to Check a SurveyJS Element Type

## Problem

I need to determine whether a given SurveyJS element is a survey, page, panel, or a specific question type.

## Solution

SurveyJS provides several ways to identify an element's type. The best approach depends on whether you need a simple category check, an exact type match, inheritance awareness, or a strict class check.

### Option 1: `is~` Boolean properties

Every SurveyJS element exposes Boolean properties that indicate its high-level category:

* [`isQuestion`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isQuestion)
* [`isPanel`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isPanel)
* [`isPage`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isPage)
* [`isSurvey`](https://surveyjs.io/form-library/documentation/api-reference/surveyelement#isSurvey)

Use these properties when you only need to know the general role of an element.

```js
if (element.isPage) {
  // The element is a page
}
```

### Option 2: `getType()` method

All survey elements implement the [`getType()`](https://surveyjs.io/form-library/documentation/api-reference/question#getType) method, which returns the element's exact type as a string (for example, `"page"`, `"panel"`, `"text"`, `"checkbox"`).

This option is useful when you need an exact type match.

```js
if (element.getType() === "page") {
  // The element is exactly a page
}
```

### Option 3: `isDescendantOf(type)` method

The [`isDescendantOf(type)`](https://surveyjs.io/form-library/documentation/api-reference/question#isDescendantOf) method checks whether an element's type matches the specified type or inherits from it. For example, since [`PageModel`](https://surveyjs.io/form-library/documentation/api-reference/page-model) extends [`PanelModel`](https://surveyjs.io/form-library/documentation/api-reference/panel-model), a page is considered a descendant of both `"page"` and `"panel"`.

```js
const page = survey.getPageByName("page1");

page.isDescendantOf("page");  // true
page.isDescendantOf("panel"); // true (PageModel extends PanelModel)
```

Use this method when type inheritance matters.

### Option 4: `instanceof` class check

You can also use the standard JavaScript [`instanceof`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof) operator. This approach requires importing the corresponding model class from `survey-core`.

The following example checks whether an element is a `PageModel` instance or a subclass of it:

```js
import { PageModel } from "survey-core";

const isPage = element instanceof PageModel;
```

This option is useful when working directly with SurveyJS model classes in strongly typed or class-aware code.

### Code Sample

```ts
import { PageModel } from "survey-core";

// `element` can be any SurveyJS element
function checkIfPage(element: any) {
  // 1. Using the Boolean property
  if (element.isPage) {
    console.log("Using isPage: This is a page.");
  }

  // 2. Using `getType()` string comparison
  if (element.getType() === "page") {
    console.log("Using getType(): This is a page.");
  }

  // 3. Using `isDescendantOf()` (type inheritance)
  if (element.isDescendantOf("page")) {
    console.log(
      "Using isDescendantOf(): This is a page or a descendant of it."
    );
  }

  // 4. Using `instanceof` (class check)
  if (element instanceof PageModel) {
    console.log("Using instanceof PageModel: This is a page.");
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

## Learn More

* [SurveyJS Form Library Documentation](https://surveyjs.io/form-library/documentation/)