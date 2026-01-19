# Create a Reusable Dynamic Matrix with Cascading Dropdowns and Lazy Loading in SurveyJS

## Problem

I need to create a reusable composite question that contains a Dynamic Matrix with two columns:

- **First column**: A Dropdown that displays regions (or any other parent category).
- **Second column**: A Dropdown that displays countries, cities, or other dependent items filtered based on the value selected in the first column of the same row.

The second Dropdown must load its choices lazily and support pagination and server-side filtering.

## Solution

To achieve this, create a [custom composite question](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types) based on a Dynamic Matrix and configure cascading Dropdowns with lazy-loaded choices:

1. **Configure the custom composite question**\
Define a configuration object that implements the [`ICustomQuestionTypeConfiguration`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration) interface. Within this object, specify a unique [`name`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#name) used to identify the component in code, a user-friendly [`title`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#title), and a [`defaultQuestionTitle`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#defaultQuestionTitle) to use for questions created using this component.

2. **Add and configure the Dynamic Matrix**\
Add a [Dynamic Matrix configuration object](https://surveyjs.io/form-library/documentation/api-reference/dynamic-matrix-table-question-model) to the [`elementsJSON`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#elementsJSON) array. Configure the matrix columns and enable lazy loading for the second column by setting the [`choicesLazyLoadEnabled`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#choicesLazyLoadEnabled) property to `true`.

1. **Register the custom composite question**\
Register the configuration object in the `ComponentCollection` so that the question becomes available in both the Form Library and Survey Creator.

1. **Lazy-load choice options**\
Handle the [`onChoicesLazyLoad`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onChoicesLazyLoad) event to load, paginate, and filter choices for the dependent Dropdown based on the selected value in the same matrix row.

1. **Preload the default value**\
When choices are loaded from a remote source, their display texts are not available until the Dropdown or Tag Box is opened. If a default value is set, you must explicitly load its display text by handling the [`onGetChoiceDisplayValue`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onGetChoiceDisplayValue) event.

### Code Sample

```js
import { ComponentCollection } from "survey-core";

// Step 1: Configure the custom composite question
const locationsConfig = {
  name: "locations",
  title: "Locations",
  defaultQuestionTitle: "Locations",
  // Step 2: Add and configure the Dynamic Matrix
  elementsJSON: [
    {
      type: "matrixdynamic",
      name: "locations-table",
      titleLocation: "hidden",
      showHeader: false,
      addRowText: "Add a Location",
      minRowCount: 1,
      columns: [
        {
          name: "region",
          cellType: "dropdown",
          placeholder: "Select a region",
          choices: ["Africa", "Americas", "Asia", "Europe", "Oceania"]          
        },
        {
          name: "country",
          cellType: "dropdown",
          placeholder: "Select a country",
          isRequired: true,
          choicesLazyLoadEnabled: true,
          choicesLazyLoadPageSize: 40,
          enableIf: "{row.region} notempty"
        }
      ]
    }
  ]
};

// Step 3: Register the custom composite question
ComponentCollection.Instance.add(locationsConfig);

// ...
// Omitted: `Survey.Model` creation
// ...

// Step 4: Lazy-load choice options
survey.onChoicesLazyLoad.add((_, options) => {
  if (options.question.name !== "country") return;

  let url = `https://surveyjs.io/api/GetCountriesPaginationRegionFilter?skip=${options.skip}&take=${options.take}`;

  const currentRowRegion = options.question.data.getQuestionByName("region")?.value;

  if (currentRowRegion) {
    url += `&region=${encodeURIComponent(currentRowRegion)}`;
  }

  if (options.filter) {
    url += `&filter=${encodeURIComponent(options.filter)}`;
  }

  sendRequest(url, (data) => {
    options.setItems(data.countries || [], data.total || 0);
  });
});

// Step 5: Preload the default value
survey.onGetChoiceDisplayValue.add((_, options) => {
  if (options.question.name !== "country") return;

  let url = `https://surveyjs.io/api/CountriesExamplePagination?skip=${options.skip}&take=${options.take}`;

  if (options.filter) {
    url += `&filter=${encodeURIComponent(options.filter)}`;
  }

  sendRequest(url, (data) => {
    options.setItems(data.countries || [], data.total || 0);
  });
});

function sendRequest(url, onloadSuccessCallback) {
  fetch(url, {
    method: "GET",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded"
    }
  })
    .then((response) => {
      if (!response.ok) {
        throw new Error(`Request failed with status ${response.status}`);
      }
      return response.json();
    })
    .then((data) => {
      onloadSuccessCallback(data);
    })
    .catch(() => {
      onloadSuccessCallback({ countries: [], total: 0 });
    });
}
```

### Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "locations",
          "name": "locations",
          "title": "Please enter your locations"
        }
      ]
    }
  ]
}
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/9jw5Rhbm9CMMByQD)

## Learn More

- [Dropdown with Lazy Loading Demo](https://surveyjs.io/form-library/examples/lazy-loading-dropdown/)