# How to create a composite question with dynamic matrix containing cascading (filtered) lazy-loading dropdowns in SurveyJS

## Problem

You want to create a **reusable composite question** in SurveyJS that contains a **dynamic matrix** with two columns:

- First column: dropdown with regions (or any parent category)
- Second column: dropdown with countries/cities/items that should be **filtered** according to the selected value in the first column of the **same row**
- Choices for the second dropdown should be loaded **lazily** from an API (with pagination & filtering support)
- The whole component should be reusable via `ComponentCollection`

## Solution

1. Register a **custom composite question** using `Survey.ComponentCollection.Instance.add()`
2. Inside the composite use `matrixdynamic` with two dropdown columns
3. Use **lazy loading** mechanism (`choicesLazyLoadEnabled: true`)
4. Fetch choices and filter them using the `survey.onChoicesLazyLoad` event
5. (Optional) Handle display value for already selected items using `onGetChoiceDisplayValue`

### Code Sample

```typescript
// 1. Register composite component
Survey.ComponentCollection.Instance.add({
  name: "registerlocations",
  defaultQuestionTitle: "Location Entries",
  elementsJSON: [
    {
      type: "matrixdynamic",
      name: "locations",
      titleLocation: "hidden",
      showHeader: false,
      allowAddRows: true,
      allowRemoveRows: true,
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
});

// 2. Handle lazy loading + row-context filtering
survey.onChoicesLazyLoad.add((sender, options) => {
  if (options.question.name !== "country") return;

  let url = `https://surveyjs.io/api/GetCountriesPaginationRegionFilter?skip=${options.skip}&take=${options.take}`;

  // Get region from the SAME row
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

// Loading display values for already saved/selected choices
survey.onGetChoiceDisplayValue.add((sender, options) => {
  if (options.question.name !== "country") return;

  let url = `https://surveyjs.io/api/CountriesExamplePagination?skip=${options.skip}&take=${options.take}`;

  if (options.filter) {
    url += `&filter=${encodeURIComponent(options.filter)}`;
  }

  sendRequest(url, (data) => {
    options.setItems(data.countries || [], data.total || 0);
  });
});

function sendRequest(url: string, onloadSuccessCallback: (data: any) => void) {
  const xhr = new XMLHttpRequest();
  xhr.open("GET", url);
  xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
  xhr.onload = () => {
    if (xhr.status === 200) {
      onloadSuccessCallback(JSON.parse(xhr.response));
    }
  };
  xhr.onerror = () => {
    onloadSuccessCallback({ countries: [], total: 0 });
  };
  xhr.send();
}
```
### Survey JSON Schema
```
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "registerlocations",
          "name": "locationGroup",
          "title": "Please enter your locations"
        }
      ]
    }
  ],
  "showQuestionNumbers": "off"
}
```
### Live demo
[View in Plunker](https://plnkr.co/edit/1Z5ymi6fvcaWUsap)

## Learn More
- [Create Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types)
- [Lazy Loading Dropdown Choices](https://surveyjs.io/form-library/examples/lazy-loading-dropdown/)
- [Matrix Dynamic Documentation](https://surveyjs.io/form-library/documentation/api-reference/dynamic-matrix-table-question-model)