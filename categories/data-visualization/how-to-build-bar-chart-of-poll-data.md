# Bar Chart for Poll Data (Custom Poll Visualizer)

## Problem

Standard chart visualizers in SurveyJS Dashboard work great for analytics, but **poll results** often look better with a clean, vote-count-focused layout:  
- Answer text + percentage + vote count  
- Horizontal bar (sparkline-style) showing the percentage  
- Simple, readable, and engaging — perfect for public polls, feedback forms, or quick surveys.

The default bar/pie charts can feel too "data-heavy" for casual poll viewing.

## Solution

Create a **custom visualizer** based on the built-in `SelectBase` visualizer.  
We keep its powerful data calculation logic (counts, percentages, sorting), but **replace the rendering function** with custom HTML that displays results in a beautiful poll-style table with inline percentage bars.

### Key Features of This Visualizer
- Shows answer, percentage, and raw vote count
- Inline horizontal bars (sparklines) for quick visual comparison
- Handles "no data" gracefully
- Works automatically with `radiogroup`, `dropdown`, and `checkbox` questions
- Fully customizable via CSS

### Code Sample

```ts
// pollVisualizer.ts
import { SelectBase, VisualizationManager, localization } from "survey-analytics";

// Custom Poll Visualizer
function PollVisualizer(question: any, data: any[], options?: any) {
  // Step 1: Custom rendering function
  function renderContent(contentContainer: HTMLElement, visualizer: any) {
    visualizer.getAnswersData().then((vizData: any) => {
      const polls = vizData.datasets;       // Vote counts
      const choices = vizData.labels;       // Answer texts
      const percentages = vizData.texts;    // Percentages as strings

      // Handle empty results
      if (polls.length === 0 || polls[0].length === 0) {
        const emptyHtml = `<p>${localization.getString("noResults")}</p>`;
        contentContainer.insertAdjacentHTML("beforeend", emptyHtml);
        return;
      }

      // Render each dataset (usually just one for single-choice questions)
      polls.forEach((poll: number[], idx: number) => {
        const table = document.createElement("table");
        table.classList.add("sa-poll-table");
        table.style.backgroundColor = visualizer.backgroundColor || "transparent";

        poll.forEach((voteCount: number, index: number) => {
          const percent = percentages[idx][index];

          const textRow = `
            <tr>
              <td class="sa-poll-table__cell">
                $$ {choices[index]} — <strong> $${percent}%</strong> ($$ {voteCount} vote $${voteCount !== 1 ? "s" : ""})
              </td>
            </tr>`;

          const barRow = `
            <tr>
              <td class="sa-poll-table__cell">
                <div class="sa-poll-sparkline">
                  <div class="sa-poll-sparkline-value" style="width: ${percent}%"></div>
                </div>
              </td>
            </tr>`;

          table.insertAdjacentHTML("beforeend", textRow + barRow);
        });

        contentContainer.appendChild(table);
      });
    });
  }

  // Step 2: Instantiate base SelectBase visualizer with custom renderer
  const visualizer = new SelectBase(
    question,
    data,
    { renderContent, dataProvider: options?.dataProvider },
    "pollVisualizer"
  );

  visualizer.answersOrder = "asc";       // Sort answers A-Z
  visualizer.showPercentages = true;     // Ensure percentages are calculated

  return visualizer;
}

// Step 3: Register as default visualizer for radiogroup questions
VisualizationManager.registerVisualizer("radiogroup", PollVisualizer, 0);

// Optional: Also support dropdown or other choice questions
VisualizationManager.registerVisualizer("dropdown", PollVisualizer, 0);

// Step 4: Localize display name in chart selector
localization.locales["en"]["visualizer_pollVisualizer"] = "Poll Results";
```

### Live Demo

[View Plunker](https://plnkr.co/edit/Ep3mv8X9tyKJ7vNz)