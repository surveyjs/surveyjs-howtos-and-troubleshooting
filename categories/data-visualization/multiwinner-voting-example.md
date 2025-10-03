# Multiwinner Voting Example

## Problem
When organizing an event like a company movie night, the venue may not accommodate all employees at once. The goal is to select multiple dates (e.g., two evenings) to maximize the number of employees who can attend at least one screening. Standard Approval Voting (AV) selects the most popular dates, but this may favor the same group of employees for both dates, leaving others uncovered. For example, both chosen dates might suit only 40% of employees, missing broader coverage.

## Solution
Multiwinner Voting Rules address this by optimizing date selection to cover more unique employees. Key features:

- **Input**: Approval ballots (checkboxes) where employees mark all suitable dates.
- **Committee Size**: Fixed number of dates (e.g., 2). Organizers can adjust this (e.g., 2 or 3 dates), and the dashboard recalculates results.
- **Algorithms**:
  - **Approval Voting (AV)**: Selects dates with the highest number of votes.
  - **Chamberlin–Courant (CC)**: Maximizes unique employee coverage.
  - **Proportional Approval Voting (PAV)**: Balances coverage and rewards employees available for multiple dates.
- **Output**: One optimal set of dates or multiple top sets if many combinations exist.

**Decision Guide**:
- Use AV for simple popularity-based selection.
- Use CC for maximum unique employee coverage.
- Use PAV to account for employees with multiple available dates.

### Code Sample
Below is a custom visualizer for SurveyJS to display voting results using AV, CC, and PAV algorithms.

```javascript
//votingAlgorythmVisualizer.js
import {
  VisualizerBase,
  VisualizationManager,
  localization,
} from "survey-analytics";
import { computeAV, computeCC, computePAV } from "./multiwinnerVotingHelper.js";

function VotingAlgorithmVisualizer(question, data, options) {
  function calculateCoverage(data) {
    const k = 2;
    const AV = computeAV(data, k);
    const CC = computeCC(data, k);
    const PAV = computePAV(data, k);
    const totalEmployees = data.length;

    function coveragePercent(selected) {
      const covered = new Set();
      data.forEach((d) => {
        if (d.availableDates.some((v) => selected.includes(v))) {
          covered.add(d);
        }
      });
      return Math.round((covered.size / totalEmployees) * 100);
    }

    return {
      coverage: { AV, CC, PAV },
      coveragePercent: {
        AV: coveragePercent(AV),
        CC: coveragePercent(CC),
        PAV: coveragePercent(PAV),
      },
    };
  }

  function renderContent(container, visualizer) {
    container.style.width = "100%";

    const { coverage, coveragePercent } = calculateCoverage(
      visualizer.surveyData
    );

    let html =
      `<div style="width:100%;">` +
      `<table style="border-collapse: collapse; width: 100%;">` +
      `<thead>` +
      `<tr>` +
      `<th style="border:1px solid #ccc; padding:4px;">Algorithm</th>` +
      `<th style="border:1px solid #ccc; padding:4px;">Selected Dates</th>` +
      `<th style="border:1px solid #ccc; padding:4px;">Employee Coverage</th>` +
      `</tr>` +
      `</thead>` +
      `<tbody>`;

    for (let alg in coverage) {
      const datesText = coverage[alg]
        .map((v) => question.choices.find((c) => c.value === v)?.text || v)
        .join(", ");

      html +=
        `<tr>` +
        `<td style="border:1px solid #ccc; padding:4px;">${alg}</td>` +
        `<td style="border:1px solid #ccc; padding:4px;">${datesText}</td>` +
        `<td style="border:1px solid #ccc; padding:4px;">${coveragePercent[alg]}%</td>` +
        `</tr>`;
    }

    html += `</tbody></table></div>`;

    html +=
      `<div style="margin-top:1em; font-style:italic;">` +
      `<p><b>Practical conclusions:</b></p>` +
      `<ul>` +
      `<li>AV (Approval Voting) → choose if you want simple popularity.</li>` +
      `<li>CC (Chamberlin–Courant) → choose if you want maximum overall coverage.</li>` +
      `<li>PAV (Proportional Approval Voting) → choose if you want to account for active employees with multiple options.</li>` +
      `</ul>` +
      `</div>`;

    container.innerHTML = html;
  }

  return new VisualizerBase(
    question,
    data,
    { renderContent },
    "voting-algorithm"
  );
}

VisualizationManager.registerVisualizer(
  "checkbox",
  VotingAlgorithmVisualizer,
  0
);

localization.locales["en"]["visualizer_voting-algorithm"] =
  "Voting Algorithm Table";
```

### Survey JSON Schema
This JSON schema defines a survey for employees to select movie screening dates.

```json
{
  "title": "Movie Screening Date Voting",
  "description": "The company is organizing TWO movie screenings. Please select all dates you are available to attend.",
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "checkbox",
          "name": "availableDates",
          "title": "Which dates are you available to attend?",
          "isRequired": true,
          "choices": [
            "Friday, Oct 10",
            "Saturday, Oct 11",
            "Friday, Oct 17",
            "Saturday, Oct 18"
          ]
        }
      ]
    }
  ]
}
```

## Learn More
- SurveyJS demos: [Implement a Custom Data Visualizer](https://surveyjs.io/dashboard/examples/custom-survey-data-visualizer/).
- Explore multiwinner voting algorithms for deeper understanding: [Chamberlin–Courant](https://en.wikipedia.org/wiki/Chamberlin%E2%80%93Courant_voting), [Proportional Approval Voting](https://en.wikipedia.org/wiki/Proportional_approval_voting).