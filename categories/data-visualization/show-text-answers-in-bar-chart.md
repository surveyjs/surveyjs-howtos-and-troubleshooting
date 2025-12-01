# Visualize Text Entry Questions as Charts in SurveyJS Dashboard

## Problem

Text entry (single-line input) questions accept open short answers that are saved in survey results as string values. SurveyJS Dashboard visualizes text entry questions as a word cloud or a table, which may not always be the most effective visualization. I want to display textual answers as **bar**, **vertical bar**, **pie**, or **doughnut charts** by counting occurrences, similar to choice-based questions.

## Solution

Create a **custom visualizer** based on the built-in `SelectBasePlotly` visualizer (used for Dropdown, Radio Button Group, and Checkboxes questions). In SurveyJS Dashboard, every visualizer consists of two core parts:

- **Data provider** &mdash; Produces aggregated values for visualization.
- **Rendering function** &mdash; Renders the visual output.

When you implement a custom data visualizer, you should base it on a built-in visualizer, but replace one or both of these parts depending on your task. The built-in `SelectBasePlotly` visualizer already can visualize data as bar, pie, and doughnut charts. You only need to implement a custom data providing function that would calculate values and format them as required by `SelectBasePlotly`.

To create a data visualizer with a custom data providing function, follow the steps below:

1. **Instantiate the visualizer**          
Create an instance of the `SelectBasePlotly` visualizer. Pass the question to be visualized, survey results, visualizer options, and your visualizer's name to the constructor.

1. **Set up available [chart types](https://surveyjs.io/dashboard/documentation/chart-types) and select the default chart type**          
Specify the visualizer's `chartTypes` array and `chartType` property.

1. **Implement a function that calculates data**           
To be used with `SelectBasePlotly`, this function should count the answers and return an array of count values. This array should have the same sort order as the array of answers. Assign this function to the visualizer's `getCalculatedValuesCore` property.

1. **Implement functions that return visualized answers and their display names**          
A visualizer must implement the `getValues` and `getLabels` functions. `getValues` should return an array of answers, `getLabels`&mdash;an array of display names. You can customize the arrays within these functions if required. In this example, both functions simply return the answer array.

1. **Register the visualizer**         
Use the `VisualizationManager.registerVisualizer(questionType, constructor, index)` method to register your custom visualizer for use with a required question type.

1. **Specify the visualizer's display name**         
This name will be displayed in the chart drop-down list. Use localization capabilities for this step.

### Code Sample

```ts
// textChartVisualizer.ts
import { SelectBasePlotly, VisualizationManager, localization } from "survey-analytics";

// A visualizer that counts textual answers and displays them in a chart
function TextChartVisualizer(question, data, options) {
  const answers = [];

  // Step 1: Instantiate the visualizer
  const visualizer = new SelectBasePlotly(
    question,
    data,
    options,
    "textChartVisualizer"
  );
  // Step 2: Set up available chart types and select the default chart type
  visualizer.chartTypes = SelectBasePlotly.types;
  visualizer.chartType = "bar";  
  // Step 3: Implement a function that calculates data
  visualizer.getCalculatedValuesCore = function () {
    const result = {};
    answers.length = 0;
    // Count each answer
    visualizer.surveyData.forEach(dataObj => {
      const answer = dataObj[visualizer.question.name];
      if (answer) {
        if (result[answer]) {
          result[answer]++;
        } else {
          result[answer] = 1;
        }
      }
    });

    // Create an array of unique answers
    answers.push.apply(answers, Object.keys(result));
    // Return an array of count values for the answers
    return [answers.map(answer => result[answer])];
  };
  // Step 4: Implement functions that return visualized answers and their display names
  visualizer.getValues = () => answers;
  visualizer.getLabels = () => answers;

  return visualizer;
}

// Step 5: Register the visualizer for Single-Line Input (Text Entry) questions and use it by default
VisualizationManager.registerVisualizer("text", TextChartVisualizer, 0);

// Step 6: Specify the visualizer's display name
localization.locales["en"]["visualizer_textChartVisualizer"] = "Texts in chart";
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/qO06ICbHC7tkAaKo)