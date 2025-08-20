# Remove Column Actions in Table View by SurveyJS

## Problem

I need to disable the following column actions in Table View:

- Search
- Drag and drop
- Sorting
- Hiding
- Moving to the detail row

## Solution

Table View actions are implemented as extensions of the grid table. To disable them, import the `TableExtensions` object from the Table View module and call its `unregisterExtension(location, extensionName)` method. Use `"column"` as the `location` and one of the following as `extensionName`:

- `"drag"` – Enables column reordering.
- `"sort"` – Enables column sorting.
- `"hide"` – Hides a column.
- `"movetodetails"` – Moves a column to the detail row.
- `"filter"` – Adds a search box to each column.

### Code Sample

```js
import { TableExtensions } from "survey-analytics/survey.analytics.tabulator";

// Remove the search box and the "Hide to Details" button 
TableExtensions.unregisterExtension("column", "hide");
TableExtensions.unregisterExtension("column", "filter");
```

## Learn More

- [Get Started with Table View](https://surveyjs.io/dashboard/documentation/set-up-table-view)
- [Table View Demo](https://surveyjs.io/dashboard/examples/export-survey-results-to-csv-files/)