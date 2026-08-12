# Bar Chart for Poll Data (Custom Poll Visualizer)

## Problem

SurveyJS Dashboard includes a set of built-in chart visualizers suitable for analytics, but their default bar and pie charts can feel too "data-heavy" for quick poll viewing. For simple polls, I prefer a layout that emphasizes vote counts by showing answer text, percentage, vote count, and a horizontal percentage bar.

## Solution

Create a **custom visualizer** based on the built-in `SelectBase` visualizer. In SurveyJS Dashboard, every visualizer consists of two core parts:

- **Data provider** &mdash; Produces aggregated values for visualization.
- **Rendering function** &mdash; Renders the visual output.

When you implement a custom data visualizer, you should base it on a built-in visualizer, but replace one or both of these parts depending on your task. For poll-style results, the built-in `SelectBase` visualizer already calculates values and returns them in a suitable format. You only need to implement a custom rendering function that would display these values as required.

To create a data visualizer with a custom rendering function, follow the steps below:

1. **Implement a rendering function**          
Within this function, configure the HTML markup that the visualizer should render.

1. **Instantiate the visualizer**          
Create an instance of a built-in visualizer (`SelectBase` in this example). Pass the question to be visualized, survey results, rendering function, and your visualizer's name to the constructor.

1. **Register the visualizer**         
Use the `VisualizationManager.registerVisualizer(questionType, constructor, index)` method to register your custom visualizer for use with a required question type.

1. **Specify the visualizer's display name**         
This name will be displayed in the chart drop-down list. Use localization capabilities for this step.

### Code Sample

```ts
// pollVisualizer.ts
import { SelectBase, VisualizationManager, localization } from "survey-analytics";

// A visualizer for poll results
function PollVisualizer(question, data, options) {
  // Step 1: Implement a rendering function
  function renderContent (contentContainer, visualizer) {
    visualizer.getAnswersData().then((vizData) => {
      const polls = vizData.datasets;
      const choices = vizData.labels;
      const percentages = vizData.texts;

      if (polls.length === 0 || polls[0].length === 0) {
        const emptyResultsHtml = `<p>` + localization.getString("noResults") + `</p>`;
        contentContainer.insertAdjacentHTML("beforeend", emptyResultsHtml);
        return;
      }

      polls.forEach((poll, idx) => {
        const tableNode = document.createElement("table");
        tableNode.classList.add("sa-poll-table");
        tableNode.style.backgroundColor = visualizer.backgroundColor;

        poll.forEach((voteCount, index) => {
          const textRow =
            `<tr>
              <td class="sa-poll-table__cell">` +
              choices[index] + " - " + percentages[idx][index] + "%" + " (" + voteCount + " votes)" + `
              </td>
            </tr>`;

          const graphRow =
            `<tr>
              <td class="sa-poll-table__cell" colspan="3">
                <div class="sa-poll-sparkline">
                  <div class="sa-poll-sparkline-value" style="width:` + percentages[idx][index] + "%" + `"></div>
                </div>
              </td>
            </tr>`;

          tableNode.insertAdjacentHTML("beforeend", textRow);
          tableNode.insertAdjacentHTML("beforeend", graphRow);
        });

        contentContainer.appendChild(tableNode);
      });
    });
  };

  // Step 2: Instantiate the visualizer
  const visualizer = new SelectBase(
    question,
    data,
    { renderContent: renderContent, dataProvider: options.dataProvider },
    "pollVisualizer"
  );
  visualizer.answersOrder = "asc";
  visualizer.showPercentages = true;
  return visualizer;
}

// Step 3: Register the visualizer for Radio Button Group questions and use it by default
SurveyAnalytics.VisualizationManager.unregisterVisualizer(
  "radiogroup",
  SurveyAnalytics.SelectBase
);
SurveyAnalytics.VisualizationManager.unregisterVisualizer(
  "radiogroup",
  SurveyAnalytics.StatisticsTable
);
SurveyAnalytics.VisualizationManager.registerVisualizer(
  "radiogroup",
  PollVisualizer,
  0,
  "pollVisualizer"
);
SurveyAnalytics.VisualizationManager.registerVisualizer(
  "pollVisualizer",
  PollVisualizer,
  0,
  "pollVisualizer"
);

// Step 4: Specify the visualizer's display name
localization.locales["en"]["visualizer_pollVisualizer"] = "Poll Visualizer";

```
Apply custom CSS:
```css
.sa-poll-table {
  width: 100%;
  font-family: SegoeUI, Arial, sans-serif;
  font-size: 14px;
  color: #404040;
  background-color: #f7f7f7;
}

.sa-poll-table__cell {
  padding: 8px;
  min-height: 34px;
}

.sa-poll-sparkline {
  min-width: 100px;
  height: 24px;
  border: 1px solid #1ab394;
  border-radius: 5px;
}

.sa-poll-sparkline-value {
  height: 100%;
  background-color: #1ab394;
}
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/6829DyUTFJtkL2aZ)