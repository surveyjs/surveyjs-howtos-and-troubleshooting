# Remove Column Actions in Table View by SurveyJS

## Problem

I need to disable the following column actions in Table View:

- Search
- Drag and drop
- Sorting
- Hiding
- Moving to the detail row

## Solution

Table View actions are implemented as extensions of the grid table. To disable them, import the `TableExtensions` object from the Table View module and call its `unregisterExtension(location, extensionName)` method.

Most column header actions use `"column"` as the `location`. The per-column search box uses a separate location: `"columnfilter"`.

Use one of the following as `extensionName`:

- `"drag"` – Enables column reordering (`location`: `"column"`).
- `"sort"` – Enables column sorting (`location`: `"column"`).
- `"hide"` – Hides a column (`location`: `"column"`).
- `"movetodetails"` – Moves a column to the detail row (`location`: `"column"`).
- `"filter"` – Adds a search box to each column (`location`: `"columnfilter"`).

### Code Sample

```js
import { TableExtensions } from "survey-analytics/survey.analytics.tabulator";

TableExtensions.unregisterExtension("column", "hide"); // Hide "Hide Column" column action
TableExtensions.unregisterExtension("column", "drag"); // Hide "Reorder Columns" column action
TableExtensions.unregisterExtension("column", "sort"); // Hide "Sort by Column" column action
TableExtensions.unregisterExtension("column", "movetodetails"); // Hide "Move to Detail" column action
TableExtensions.unregisterExtension("columnfilter", "filter"); // Hide column search box
```

## Live Demo

[Open in Plunker](https://plnkr.co/edit/D9J3sGzdyDkjhHrG)

## Learn More

- [Get Started with Table View](https://surveyjs.io/dashboard/documentation/set-up-table-view)
- [Table View Demo](https://surveyjs.io/dashboard/examples/export-survey-results-to-csv-files/)
