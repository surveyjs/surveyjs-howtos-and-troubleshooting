# Implement a Custom Data Visualizer in SurveyJS Dashboard

## Problem

SurveyJS Dashboard includes powerful built-in visualizers (bar, pie, histogram, gauge, word cloud, etc.), but sometimes you need **completely custom rendering** for specific question types:

- Display a **Matrix** question as a clean, styled HTML table (instead of the default chart)
- Show **Multiple Textboxes** with calculated averages, highest/lowest values, etc.

Built-in visualizers may not offer the exact layout or calculations you need.

## Solution

Create **custom visualizers** by extending existing ones (`VisualizerBase`, `Matrix`, etc.). Replace only the **rendering function** and reuse their robust data calculation logic. This approach gives you full control over HTML/CSS while keeping performance and data accuracy.

To create a custom visualizer for SurveyJS Dashboard, follow the steps below:

1. **Implement a rendering function**          
Within this function, configure the HTML markup that the visualizer should render.

2. **Instantiate the visualizer**          
Create an instance of a built-in visualizer (`VisualizerBase` and `Matrix` in this example). Pass the question to be visualized, survey results, rendering function, and your visualizer's name to the constructor.

3. **Register the visualizer**         
Use the `VisualizationManager.registerVisualizer(questionType, constructor, index)` method to register your custom visualizer for use with a required question type.

4. **Specify the visualizer's display name**         
This name will be displayed in the chart drop-down list. Use localization capabilities for this step.

### Code Sample

```ts
// customVisualizers.ts
import { VisualizerBase, Matrix, VisualizationManager, localization } from "survey-analytics";

// A visualizer that displays matrix results in the table form
function MatrixTableVisualizer (question, data, options) {
  function renderHeader (visualizer) {
    let thHtml = "";
    visualizer.valuesSource().forEach(dataItem => {
      thHtml += `<th>` + dataItem.text + `</th>`
    });

    return `
      <tr>
        <th></th> ` +
        thHtml + `
      </tr>
    `;
  }

  async function renderRows (visualizer) {
    let data = await visualizer.getCalculatedValues();
    const rows = visualizer.getValues();
    const columns = visualizer.getSeriesValues();

    let rowsHtml = "";
    visualizer.getSeriesLabels().forEach((label, rowIndex) => {
      const sum = data[rowIndex].reduce((a, b) => a + b, 0);
      let tdHtml = "<td>" + label + "</td>";
      visualizer.valuesSource().forEach(dataItem => {
        const columnIndex = rows.indexOf(dataItem.value);
        const voteCount = data[rowIndex][columnIndex];
        tdHtml += `
          <td>` +
            voteCount + " | " + Math.round((voteCount / sum) * 100) + "%"; + `
          </td>
        `;
      });

      rowsHtml += "<tr>" + tdHtml + "</tr>";
    });

    return rowsHtml;
  }

  // Step 1: Implement a rendering function
  function renderContent (contentContainer, visualizer) {
    const headerHtml = renderHeader(visualizer);
    renderRows(visualizer).then((rowsHtml) => {
      const tableHtml = `
        <table class="sa__matrix-table">` +
          headerHtml +
          rowsHtml + `
        </table>
      `;
      contentContainer.insertAdjacentHTML("beforeend", tableHtml);
      visualizer.onUpdate();
    });
  };

  // Step 2: Instantiate the visualizer
  return new Matrix(
    question,
    data,
    {
      renderContent: renderContent, dataProvider: options.dataProvider
    },
    "matrix-table"
  );
}

// A visualizer that calculates the average lowest and highest prices and displays them
function AvgPriceLimitVisualizer (question, data, options) {
  function getData (visualizer) {
    let result = [0, 0];
    let counter = 0;
    visualizer.surveyData.forEach(dataObj => {
      const multipleTextValue = dataObj[visualizer.question.name];
      if (!!multipleTextValue) {
        result[0] += multipleTextValue.leastamount;
        result[1] += multipleTextValue.mostamount;
        counter++;
      }
    });

    result[0] = result[0] / counter;
    result[1] = result[1] / counter;

    return result;
  };

  // Step 1: Implement a rendering function
  function renderContent (contentContainer, visualizer) {
    const dataToRender = getData(visualizer);
    const minHtml = `
      <div>
        <span>
          Avg. lowest price: ` + dataToRender[0] + `
        </span>
      </div>
    `;
    const maxHtml = `
      <div>
        <span>
          Avg. highest price: ` + dataToRender[1] + `
        </span>
      </div>
    `;
    contentContainer.insertAdjacentHTML("beforeend", minHtml + maxHtml);
  };

  // Step 2: Instantiate the visualizer
  return new VisualizerBase(
    question,
    data,
    { renderContent: renderContent, dataProvider: options.dataProvider },
    "avg-price-limit"
  );
}

// Step 3: Register the visualizers and use them by default
VisualizationManager.registerVisualizer("matrix", MatrixTableVisualizer, 0);
VisualizationManager.registerVisualizer("multipletext", AvgPriceLimitVisualizer, 0);

// Step 4: Specify the visualizers' display names
localization.locales["en"]["visualizer_matrix-table"] = "Table";
localization.locales["en"]["visualizer_avg-price-limit"] = "Avg. Lowest and Highest Price";
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/euAlF27O2NQFb4N8)