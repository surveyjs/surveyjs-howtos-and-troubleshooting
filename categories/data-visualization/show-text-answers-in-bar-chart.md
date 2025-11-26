# Visualize Text Entry Questions as Charts

## Problem

Text entry (single-line input) questions in SurveyJS collect open-ended string responses (e.g., zip codes, names, comments). By default, SurveyJS Dashboard displays these as a **word cloud** or **table**, which may not always be the most effective visualization — especially when answers follow a pattern or have limited unique values (like zip codes).

You want to display textual answers as **bar**, **vertical bar**, **pie**, or **doughnut charts** by counting occurrences — just like choice-based questions.

## Solution

Create a **custom visualizer** based on the built-in `SelectBasePlotly` visualizer (used for dropdown, radio, and checkbox questions). This allows reuse of all chart types (bar, pie, etc.) while replacing only the **data calculation logic** to count text occurrences.

### Steps Overview

1. Instantiate `SelectBasePlotly` as the base visualizer.
2. Define supported chart types and default type.
3. Override `getCalculatedValuesCore` to count text answers.
4. Implement `getValues()` and `getLabels()` to return answer keys.
5. Register the visualizer for `"text"` question type (with priority 0 to make it default).
6. Localize the visualizer name for the UI.

### Code Sample

```ts
// textChartVisualizer.ts
import { DataProvider, SelectBasePlotly, VisualizationManager, localization } from "survey-analytics";

// Custom visualizer: counts textual answers and displays them as bar/pie/doughnut charts
function TextChartVisualizer(question: any, data: any[], options?: any) {
  const answers: string[] = [];

  // Step 1: Create base SelectBasePlotly visualizer
  const visualizer = new SelectBasePlotly(
    question,
    data,
    options,
    "textChartVisualizer"
  );

  // Step 2: Enable all standard chart types, default to bar
  visualizer.chartTypes = SelectBasePlotly.types;
  visualizer.chartType = "bar";

  // Step 3: Count occurrences of each text answer
  visualizer.getCalculatedValuesCore = function () {
    const result: Record<string, number> = {};
    answers.length = 0;

    visualizer.surveyData.forEach((dataObj: any) => {
      const answer = dataObj[visualizer.question.name];
      if (answer) {
        result[answer] = (result[answer] || 0) + 1;
      }
    });

    // Build sorted list of unique answers
    answers.push(...Object.keys(result));

    // Return counts in the same order as answers
    return [answers.map(answer => result[answer])];
  };

  // Step 4: Provide values and labels (same in this case)
  visualizer.getValues = () => answers;
  visualizer.getLabels = () => answers;

  return visualizer;
}

// Step 5: Register as default visualizer for text questions
VisualizationManager.registerVisualizer("text", TextChartVisualizer, 0);

// Step 6: Add display name (shown in chart type selector)
localization.locales["en"]["visualizer_textChartVisualizer"] = "Texts in Chart";
```
[View Plunker](https://plnkr.co/edit/qO06ICbHC7tkAaKo)