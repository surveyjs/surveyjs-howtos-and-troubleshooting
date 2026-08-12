# Implement a Custom Data Visualizer in SurveyJS Dashboard

## Problem

SurveyJS Dashboard includes powerful built-in visualizers (bar, pie, histogram, gauge, word cloud, etc.), but sometimes you need completely custom rendering for specific question types:

- Display a Matrix question as a clean, styled HTML table (instead of the default chart)
- Show Multiple Textboxes with calculated averages, highest/lowest values, etc.

Built-in visualizers may not offer the exact layout or calculations you need.

## Solution

Create custom visualizers by extending existing ones (`VisualizerBase`, `Matrix`, etc.). Replace only the rendering function and reuse their data calculation logic. This approach gives you full control over HTML/CSS while keeping performance and data accuracy.

The registration pattern matches other Dashboard v3 custom visualizers: pass `dataProvider`, unregister defaults you want to replace, register for both the question type and the visualizer name, then render with `Dashboard`.

To create a custom visualizer for SurveyJS Dashboard, follow the steps below:

1. Implement a rendering function that builds the HTML markup.
2. Instantiate a built-in visualizer (`VisualizerBase` or `Matrix` in this example). Pass the question, survey results, options (including `renderContent` and `dataProvider`), and your visualizer name.
3. Unregister default visualizers for the target question types, then register yours with [`VisualizationManager.registerVisualizer`](https://surveyjs.io/dashboard/documentation/api-reference/visualizationmanager#registerVisualizer).
4. Specify the visualizer's display name via localization.
5. Create a `Dashboard` instance and call `render`.

### Code Sample

```ts
// customVisualizers.ts
import {
  VisualizerBase,
  Matrix,
  VisualizationManager,
  localization,
  WordCloud,
  Text,
} from "survey-analytics";

// A visualizer that displays matrix results in table form
function MatrixTableVisualizer(question, data, options) {
  options = options || {};

  const cellStyle =
    "border:1px solid #ccc; padding:8px 10px; vertical-align:middle;";
  const headerCellStyle =
    cellStyle +
    " background:#f5f5f5; font-weight:600; text-align:center; white-space:nowrap;";
  const labelCellStyle = cellStyle + " text-align:left; white-space:normal;";
  const valueCellStyle =
    cellStyle + " text-align:center; white-space:nowrap;";

  function renderHeader(visualizer) {
    let thHtml = "";
    visualizer.valuesSource().forEach((dataItem) => {
      thHtml += `<th style="` + headerCellStyle + `">` + dataItem.text + `</th>`;
    });

    return `
      <tr>
        <th style="` +
      headerCellStyle +
      `"></th>` +
      thHtml + `
      </tr>
    `;
  }

  async function renderRows(visualizer) {
    const calculated = await visualizer.getCalculatedValues();
    // Dashboard v3 returns { data, values, series }
    const tableData = calculated.data || calculated;
    const rows = visualizer.getValues();

    let rowsHtml = "";
    visualizer.getSeriesLabels().forEach((label, rowIndex) => {
      const rowCounts = tableData[rowIndex] || [];
      const sum = rowCounts.reduce((a, b) => a + b, 0) || 1;
      let tdHtml =
        `<td style="` + labelCellStyle + `">` + label + `</td>`;

      visualizer.valuesSource().forEach((dataItem) => {
        const columnIndex = rows.indexOf(dataItem.value);
        const voteCount = rowCounts[columnIndex] || 0;
        tdHtml +=
          `<td style="` +
          valueCellStyle +
          `">` +
          voteCount +
          " | " +
          Math.round((voteCount / sum) * 100) +
          `%</td>`;
      });

      rowsHtml += "<tr>" + tdHtml + "</tr>";
    });

    return rowsHtml;
  }

  // Step 1: Implement a rendering function
  function renderContent(contentContainer, visualizer) {
    contentContainer.style.width = "100%";
    contentContainer.style.overflowX = "auto";

    const headerHtml = renderHeader(visualizer);
    renderRows(visualizer).then((rowsHtml) => {
      const tableHtml =
        `<table class="sa__matrix-table" style="border-collapse:collapse; width:100%; min-width:640px;">` +
        headerHtml +
        rowsHtml +
        `</table>`;
      contentContainer.insertAdjacentHTML("beforeend", tableHtml);
      if (typeof visualizer.onUpdate === "function") {
        visualizer.onUpdate();
      }
    });
  }

  // Step 2: Instantiate the visualizer
  return new Matrix(
    question,
    data,
    {
      renderContent: renderContent,
      dataProvider: options.dataProvider,
    },
    "matrix-table"
  );
}

// A visualizer that calculates the average lowest and highest prices and displays them
function AvgPriceLimitVisualizer(question, data, options) {
  options = options || {};

  function getData(visualizer) {
    let result = [0, 0];
    let counter = 0;

    visualizer.surveyData.forEach((dataObj) => {
      const multipleTextValue = dataObj[visualizer.question.name];
      if (!!multipleTextValue) {
        result[0] += Number(multipleTextValue.leastamount) || 0;
        result[1] += Number(multipleTextValue.mostamount) || 0;
        counter++;
      }
    });

    if (counter > 0) {
      result[0] = result[0] / counter;
      result[1] = result[1] / counter;
    }

    return result;
  }

  // Step 1: Implement a rendering function
  function renderContent(contentContainer, visualizer) {
    const dataToRender = getData(visualizer);
    const rowStyle = "padding:6px 0; line-height:1.5;";
    const minHtml =
      `<div style="` +
      rowStyle +
      `"><span>Avg. lowest price: ` +
      dataToRender[0].toFixed(2) +
      `</span></div>`;
    const maxHtml =
      `<div style="` +
      rowStyle +
      `"><span>Avg. highest price: ` +
      dataToRender[1].toFixed(2) +
      `</span></div>`;
    contentContainer.insertAdjacentHTML("beforeend", minHtml + maxHtml);
  }

  // Step 2: Instantiate the visualizer
  return new VisualizerBase(
    question,
    data,
    { renderContent: renderContent, dataProvider: options.dataProvider },
    "avg-price-limit"
  );
}

// Step 3: Replace defaults and register the custom visualizers
VisualizationManager.unregisterVisualizer("matrix", Matrix);
VisualizationManager.unregisterVisualizer("multipletext", WordCloud);
VisualizationManager.unregisterVisualizer("multipletext", Text);

VisualizationManager.registerVisualizer(
  "matrix",
  MatrixTableVisualizer,
  0,
  "matrix-table"
);
VisualizationManager.registerVisualizer(
  "matrix-table",
  MatrixTableVisualizer,
  0,
  "matrix-table"
);

VisualizationManager.registerVisualizer(
  "multipletext",
  AvgPriceLimitVisualizer,
  0,
  "avg-price-limit"
);
VisualizationManager.registerVisualizer(
  "avg-price-limit",
  AvgPriceLimitVisualizer,
  0,
  "avg-price-limit"
);

// Step 4: Specify the visualizers' display names
localization.locales["en"]["visualizer_matrix-table"] = "Table";
localization.locales["en"]["visualizer_avg-price-limit"] =
  "Avg. Lowest and Highest Price";
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

  const loadingIndicator = document.getElementById("loadingIndicator");
  if (loadingIndicator) {
    loadingIndicator.style.display = "none";
  }
  dashboard.render("surveyDashboardContainer");
}, 1000);
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/zJOxOoGm6KzFyjNw)

## Learn More

- [Implement a Custom Data Visualizer](https://surveyjs.io/dashboard/examples/custom-survey-data-visualizer/)
- [Get Started with SurveyJS Dashboard](https://surveyjs.io/dashboard/documentation/get-started)
