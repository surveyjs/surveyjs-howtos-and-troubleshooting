# Dynamically Change Answer Options Based on the Survey Locale

## Problem

I want to display a different set of answer choices depending on the currently selected survey language.

## Solution

Apply `visibleIf` expressions to individual choice options and evaluate them against the survey's current locale. Use `{$survey.locale}` to access the survey's [`locale`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#locale) property in expressions.

Be aware that when the default locale is active, the `locale` property contains an empty string (`""`). Your expressions should explicitly account for this behavior.

### Survey JSON Schema

The following example shows how to conditionally display a choice option based on the survey locale. In this configuration, the Winter Fuel Payment option is shown only when the default locale is used or when English is explicitly selected.

```json
{
  "pages": [
    {
      "name": "page1",
      "title": "Select a locale",
      "description": "The set of available choices changes based on the selected locale.",
      "elements": [
        {
          "type": "checkbox",
          "name": "eligibleFor",
          "title": {
            "default": "Are you eligible for?",
            "es": "¿Es usted elegible para?"
          },
          "choices": [
            {
              "value": "credit",
              "text": {
                "default": "Universal Credit",
                "es": "Ingreso Mínimo Vital"
              }
            },
            {
              "value": "jobseek",
              "text": {
                "default": "Jobseeker's Allowance",
                "es": "Prestación por Desempleo"
              }
            },
            {
              "value": "winter_fuel",
              "text": "Winter Fuel Payment",
              "visibleIf": "{$survey.locale} empty or {$survey.locale} = 'en'"
            }
          ]
        }
      ]
    }
  ]
}
```


### Code Sample

The following example demonstrates how to add a language selector that allows respondents to switch the survey language at runtime:

```js
import { createDropdownActionModel, ComputedUpdater } from "survey-core";

// ...
// Omitted: `Survey.Model` creation
// ...

// Add a language selector to the page title
survey.onGetPageTitleActions.add((_, options) => {
  const locales = [
    { id: "en", title: "English" },
    { id: "es", title: "Español" }
  ];

  options.titleActions.push(
    createDropdownActionModel(
      {
        id: "language-selector",
        title: new ComputedUpdater(() => {
          const loc = survey.locale;
          if (!loc || loc === "en") {
            return "English";
          }
          // Find matching locale title
          const locale = locales.find(l => l.id === loc);
          return locale ? locale.title : "English";
        })
      },
      {
        items: locales,
        allowSelection: true,
        onSelectionChanged: (locale) => {
          // Use an empty string for the default locale (usually English)
          survey.locale = locale && locale.id === "en" ? "" : (locale ? locale.id : "");
          // Required in SurveyJS v2.5.5 and earlier to re-evaluate expressions
          survey.runExpressions();
        }
      }
    )
  );
});
```

### Live Demo

[Open in Plunker](https://plnkr.co/edit/VWFwlIM6c1VqLRVk)

## Learn More

* [Localization and Globalization in SurveyJS Form Library](https://surveyjs.io/form-library/documentation/survey-localization)
* [Expressions](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#expressions)
* [Context Actions in Element Titles](https://surveyjs.io/form-library/examples/modify-titles-of-survey-elements/)