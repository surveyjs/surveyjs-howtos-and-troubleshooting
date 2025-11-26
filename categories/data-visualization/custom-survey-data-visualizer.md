# Implement a Custom Data Visualizer

## Problem

SurveyJS Dashboard includes powerful built-in visualizers (bar, pie, histogram, gauge, word cloud, etc.), but sometimes you need **completely custom rendering** for specific question types:

- Display a **Matrix** question as a clean, styled HTML table (instead of the default chart)
- Show **Multiple Textboxes** with calculated averages, highest/lowest values, etc.

Built-in visualizers may not offer the exact layout or calculations you need.

## Solution

Create **custom visualizers** by extending existing ones (`VisualizerBase`, `Matrix`, etc.) and replacing only the **rendering function** — while reusing their robust data calculation logic.

This approach gives you full control over HTML/CSS while keeping performance and correctness.

### Step-by-Step Guide

1. **Implement a custom `renderContent` function** — full control over markup  
2. **Instantiate a base visualizer** (`VisualizerBase`, `Matrix`, etc.) and inject your renderer  
3. **Register the visualizer** for specific question types  
4. **Localize the display name** for the chart selector dropdown

### Code Sample

```ts
// customVisualizers.ts
import { VisualizerBase, Matrix, VisualizationManager, localization } from "survey-analytics";

// 1. Custom Table Visualizer for Matrix Questions
function MatrixTableVisualizer(question: any, data: any[], options?: any) {
  function renderContent(container: HTMLElement, visualizer: any) {
    const matrixData = visualizer.getCalculatedValues();
    const rows = question.rows;
    const columns = question.columns;

    let html = `<table class="sa-matrix-table">
      <thead><tr><th></th>`;
    
    columns.forEach((col: any) => {
      html += `<th>${col.text || col.value}</th>`;
    });
    html += `</tr></thead><tbody>`;

    rows.forEach((row: any) => {
      html += `<tr><td class="sa-matrix-row-label">${row.text || row.value}</td>`;
      columns.forEach((col: any) => {
        const cellValue = matrixData[row.value]?.[col.value] || 0;
        html += `<td class="sa-matrix-cell">${cellValue}</td>`;
      });
      html += `</tr>`;
    });

    html += `</tbody></table>`;
    container.innerHTML = html;
  }

  const visualizer = new Matrix(question, data, { renderContent }, "matrixTable");
  return visualizer;
}

// 2. Multiple Textboxes: Show Average, Min, Max
function MultipleTextStatsVisualizer(question: any, data: any[], options?: any) {
  function renderContent(container: HTMLElement, visualizer: any) {
    const stats = visualizer.getCalculatedValues(); // Array of objects
    let html = `<div class="sa-multitext-stats">`;

    question.items.forEach((item: any, idx: number) => {
      const values = stats.map((row: any) => row[item.name]).filter((v: any) => v != null);
      if (values.length === 0) return;

      const avg = (values.reduce((a: number, b: number) => a + b, 0) / values.length).toFixed(1);
      const min = Math.min(...values);
      const max = Math.max(...values);

      html += `
        <div class="sa-multitext-item">
          <div class="sa-multitext-label">${item.title || item.name}</div>
          <div class="sa-multitext-values">
            <span>Avg: <strong>${avg}</strong></span>
            <span>Min: <strong>${min}</strong></span>
            <span>Max: <strong>${max}</strong></span>
          </div>
        </div>`;
    });

    html += `</div>`;
    container.innerHTML = html;
  }

  const visualizer = new VisualizerBase(question, data, { renderContent }, "multitextStats");
  return visualizer;
}

// 3. Register visualizers
VisualizationManager.registerVisualizer("matrix", MatrixTableVisualizer, 0);
VisualizationManager.registerVisualizer("multipletext", MultipleTextStatsVisualizer, 0);

// 4. Localize display names
localization.locales["en"]["visualizer_matrixTable"] = "Table View";
localization.locales["en"]["visualizer_multitextStats"] = "Statistics Summary";
```

### Live Demo

[View Plunker](https://plnkr.co/edit/5Nz65FnimYLoYSMd)