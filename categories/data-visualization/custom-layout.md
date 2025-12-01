# Disable the Layout Engine in SurveyJS Dashboard

## Problem

SurveyJS Dashboard includes a built-in layout engine that automatically arranges dashboard charts on the page to make efficient use of screen space. In some cases, you may want full control over the layout (for example, to stack all charts vertically in a fixed order) and prevent automatic repositioning or user-driven drag-and-drop rearrangement.

## Solution

To disable the built-in layout engine, follow these steps:

1. Create a custom layout engine that implements the required interface (`start`, `stop`, `update`, `destroy`) but leaves all methods empty.
2. Pass an instance of this custom engine to the [`VisualizationPanel`](https://surveyjs.io/dashboard/documentation/api-reference/visualization-panel)'s [`layoutEngine`](https://surveyjs.io/dashboard/documentation/visualizationpanel#layoutEngine) option.
3. Disable the [`allowDynamicLayout`](https://surveyjs.io/dashboard/documentation/visualizationpanel#allowDynamicLayout) option to hide the drag-and-drop handle since reordering is no longer supported.

### Code Sample

```ts
import { Model } from "survey-core";
import { VisualizationPanel } from "survey-analytics";

// ...
// Omitted: `Survey.Model` creation and results loading
// ...

function LayoutEngine() {
  return {
    start: function () { },
    stop: function () { },
    update: function () { },
    destroy: function () { }
  };
}

const vizPanel = new VisualizationPanel(
  survey.getAllQuestions(),
  surveyResults,
  { 
    layoutEngine: new LayoutEngine(), 
    allowDynamicLayout: false 
  }
);

vizPanel.render("surveyDashboardContainer");
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/ITfHQ1u7nWjoaIED)