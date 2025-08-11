# Add Tooltips to Survey Questions

## Problem

I want to place toggle buttons next to question titles that display help text when clicked or hovered.

## Solution

In SurveyJS, the most accessible way to provide additional guidance is to use **question descriptions**. A description appears below the question title and serves the same purpose as a tooltip&mdash;helping respondents answer the question correctly.

```json
{
  "type": "text",
  "name": "full-name",
  "title": "What is your name?",
  "description": "Please enter your full legal name as it appears in official documents"
}
```

We recommend using descriptions instead of tooltips because tooltips are not reliably available on mobile devices and can reduce accessibility.

If you still want to use tooltips, you can implement them with **custom title actions**. See the following demos for details:

- [Form Library: Context Actions in Element Titles](https://surveyjs.io/form-library/examples/modify-titles-of-survey-elements/)
- [Survey Creator: Customize the Designer and Preview Tabs](https://surveyjs.io/survey-creator/examples/customize-form-builder-ui/)
