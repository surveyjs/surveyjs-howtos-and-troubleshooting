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
VisualizationManager.registerVisualizer("radiogroup", PollVisualizer, 0);

// Step 4: Specify the visualizer's display name
localization.locales["en"]["visualizer_pollVisualizer"] = "Poll Visualizer";

```



### Live Demo

[Open in Plunker](https://plnkr.co/edit/Ep3mv8X9tyKJ7vNz)