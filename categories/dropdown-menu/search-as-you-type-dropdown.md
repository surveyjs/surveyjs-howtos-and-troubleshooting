# Implement Search-as-You-Type Dropdown with API Calls

## Problem
How to create a dropdown that searches an API as the user types and populates results dynamically?

## Solution
Enable the [Lazy Loading](https://surveyjs.io/form-library/examples/lazy-loading-dropdown/) feature and use the [`SurveyModel.onChoicesLazyLoad`](https://surveyjs.io/form-library/documentation/surveymodel#onChoicesLazyLoad) event to implement search-as-you-type functionality with API calls.

Within the `onChoicesLazyLoad` function, use `options.filter` to detect when the user is typing and call `options.setItems([], 0)` for an empty initial state. When a user specifies a search string, send a request to your API endpoint to fetch choices and use `options.setItems(data.results, data.total)` to populate the dropdown list. To configure a debounce value (a delay time before the API call is made), use the [`settings.dropdownSearchDelay`](https://surveyjs.io/form-library/documentation/api-reference/settings#dropdownSearchDelay) setting.

## Code Sample
```js
// Set debounce delay (in milliseconds)
survey.settings.dropdownSearchDelay = 500;

// Handle lazy loading with API calls
survey.onChoicesLazyLoad.add((_, options) => {  
 if (!options.filter) {
   // Show empty dropdown initially
   options.setItems([], 0);
 } else {
   // Make API call with user's search term
   const url = `https://api.example.com/search?q=${options.filter}&skip=${options.skip}&take=${options.take}`;
   sendRequest(url, (data) => { 
     options.setItems(data.results, data.total); 
   });   
 }
});
```
