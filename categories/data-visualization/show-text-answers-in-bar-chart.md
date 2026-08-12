# Visualize Text Entry Questions as Charts in SurveyJS Dashboard

## Problem

Text entry (single-line input) questions accept open short answers that are saved in survey results as string values. SurveyJS Dashboard visualizes text entry questions as a word cloud or a table, which may not always be the most effective visualization. I want to display textual answers as bar, vertical bar, pie, or doughnut charts by counting occurrences, similar to choice-based questions.

## Solution

Create a custom visualizer based on the built-in [`SelectBase`](https://surveyjs.io/dashboard/documentation/api-reference/selectbase) visualizer (used for Dropdown, Radio Button Group, Checkboxes, and similar questions). In SurveyJS Dashboard v3, charts are rendered with [Chart.js](https://www.chartjs.org/) by default. Every visualizer consists of two core parts:

- Data provider &mdash; Produces aggregated values for visualization.
- Rendering function &mdash; Renders the visual output.

When you implement a custom data visualizer, you should base it on a built-in visualizer, but replace one or both of these parts depending on your task. The built-in `SelectBase` visualizer already can visualize data as bar, pie, and doughnut charts. You only need to implement a custom data providing function that calculates values and formats them as required by Dashboard v3.

To create a data visualizer with a custom data providing function, follow the steps below:

1. Instantiate the visualizer.
   Create an instance of the `SelectBase` visualizer. Pass the question, survey results, options (including `dataProvider`), and your visualizer's name to the constructor.

2. Set up available [chart types](https://surveyjs.io/dashboard/documentation/chart-types) and select the default chart type.
   Specify the visualizer's `chartTypes` array and `chartType` property. Use `chartTypes.selectBase` from `survey-analytics`.

3. Implement a function that calculates data.
   Assign it to `getCalculatedValuesCore`. In Dashboard v3 this function must return an object of the form `{ data: [counts], values: answers }`, not a bare array of counts.

4. Implement functions that return visualized answers and their display names.
   A visualizer must implement `getValues` and `getLabels`. `getValues` should return an array of answers, `getLabels`&mdash;an array of display names. In this example, both functions return the answer array.

5. Register the visualizer alongside the built-in ones.
   Call [`VisualizationManager.registerVisualizer`](https://surveyjs.io/dashboard/documentation/api-reference/visualizationmanager#registerVisualizer) for the `"text"` question type and for the visualizer name. Do not unregister `WordCloud` or `Text`.

   In Dashboard v3 there is no separate drop-down for switching visualizers. Chart types and alternative visualizers share one toolbar control. With this registration, that control lists bar / vertical bar / pie / doughnut (from the custom chart visualizer), plus Word cloud and Texts in table. A second control still provides answer order (ascending / descending). The localized name `"Texts in chart"` is used when the chart visualizer has no `chartTypes`; when `chartTypes` are set, the chart type labels are shown instead.

6. Specify the visualizer's display name via localization (used if `chartTypes` is empty).

7. Create a `Dashboard` instance and call `render`.

### Code Sample

```ts
// textChartVisualizer.ts
import {
  SelectBase,
  VisualizationManager,
  localization,
  chartTypes,
} from "survey-analytics";

// A visualizer that counts textual answers and displays them in a chart
function TextChartVisualizer(question, data, options) {
  const answers = [];
  options = options || {};

  // Step 1: Instantiate the visualizer
  const visualizer = new SelectBase(
    question,
    data,
    { ...options, dataProvider: options.dataProvider },
    "textChartVisualizer"
  );

  // Step 2: Set up available chart types and select the default chart type
  visualizer.chartTypes = (chartTypes.selectBase || [
    "bar",
    "vbar",
    "pie",
    "doughnut",
  ]).slice();
  visualizer.chartType = "bar";

  // Step 3: Implement a function that calculates data
  visualizer.getCalculatedValuesCore = function () {
    const result = {};
    answers.length = 0;

    // Count each answer
    visualizer.surveyData.forEach((dataObj) => {
      const answer = dataObj[visualizer.question.name];
      if (answer) {
        result[answer] = (result[answer] || 0) + 1;
      }
    });

    // Create an array of unique answers
    answers.push.apply(answers, Object.keys(result));
    const counts = answers.map((answer) => result[answer]);

    // Dashboard v3 expects { data, values } (not a bare [counts] array)
    return {
      data: [counts],
      values: answers.slice(),
    };
  };

  // Step 4: Implement functions that return visualized answers and their display names
  visualizer.getValues = () => answers;
  visualizer.getLabels = () => answers;

  return visualizer;
}

// Step 5: Register the visualizer for Text Entry questions (keeps WordCloud and Text)
VisualizationManager.registerVisualizer(
  "text",
  TextChartVisualizer,
  0,
  "textChartVisualizer"
);
VisualizationManager.registerVisualizer(
  "textChartVisualizer",
  TextChartVisualizer,
  0,
  "textChartVisualizer"
);

// Step 6: Specify the visualizer's display name
localization.locales["en"]["visualizer_textChartVisualizer"] = "Texts in chart";
```

#### Render with Dashboard

```js
const survey = new Survey.Model(json);

setTimeout(() => {
  const dashboard = new SurveyAnalytics.Dashboard({
    questions: survey.getAllQuestions(),
    data: dataFromServer,
    allowDynamicLayout: false,
    allowHideQuestions: false
  });

  dashboard.applyTheme(SurveyTheme.MonochromeLight);
  dashboard.render("surveyDashboardContainer");
}, 1000);
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/5aRRwfNfUp1P45SF)

## Learn More

- [Implement a Custom Data Visualizer](https://surveyjs.io/dashboard/examples/custom-survey-data-visualizer/)
- [Get Started with SurveyJS Dashboard](https://surveyjs.io/dashboard/documentation/get-started)
- [Chart Types](https://surveyjs.io/dashboard/documentation/chart-types)
