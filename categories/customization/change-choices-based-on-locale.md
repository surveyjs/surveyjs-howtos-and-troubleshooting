# Dynamically change number of choices based on survey language/locale

## Problem

How to display a **different number of answer options** depending on the selected survey language.

Common real-world examples:

- Social benefits or allowances available only in certain countries/languages
- Region-specific options
- Intentionally omitting choices in one language version

## Solution

To achieve this, follow these steps:

1. Use localized text for choice labels (`text` as object with language keys).
2. To control visibility of individual choices based on the survey's locale, define the `visibleIf` property on each choice and use `{$survey.locale}` to reference the [`survey.locale`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#locale) property in expressions.
3. Add a language switcher to allow users dynamically change a survey locale.

### Code Sample

The following code adds a language dropdown to the page header.

```typescript
survey.onGetPageTitleActions.add(function (sender, options) {
    var localeItems = [
        { id: "en", title: "English" },
        { id: "es", title: "Español" }
        // Add more languages here as needed, e.g.:
        // { id: "fr", title: "Français" }
    ];

    options.titleActions.push(
        Survey.createDropdownActionModel(
            {
                id: "language-selector",
                title: new Survey.ComputedUpdater(function () {
                    var loc = survey.locale;

                    if (!loc || loc === "en") {
                        return "English";
                    }

                    // Find matching locale title (safe for older environments)
                    for (var i = 0; i < localeItems.length; i++) {
                        if (localeItems[i].id === loc) {
                            return localeItems[i].title;
                        }
                    }

                    return "English"; // fallback
                })
            },
            {
                items: localeItems,
                allowSelection: true,
                onSelectionChanged: function (item) {
                    // Set locale: "" = default (usually English), otherwise selected id
                    survey.locale = item && item.id === "en" ? "" : (item ? item.id : "");
                    survey.runExpressions(); // Important: re-evaluate visibleIf expressions
                }
            }
        )
    );
});
```

### Survey JSON Schema
```json
{
  "pages": [
    {
      "name": "benefits",
      "title": "Benefits & Support",
      "elements": [
        {
          "type": "checkbox",
          "name": "eligibleFor",
          "title": {
            "default": "Which benefits are you currently receiving or applying for?",
            "es": "¿De qué prestaciones recibe o está solicitando actualmente?",
            "fr": "Quelles prestations recevez-vous ou demandez-vous actuellement ?"
          },
          "choices": [
            {
              "value": "uc",
              "text": {
                "default": "Universal Credit",
                "es": "Ingreso Mínimo Vital",
                "fr": "Revenu de Solidarité Active (RSA)"
              }
            },
            {
              "value": "jsa",
              "text": {
                "default": "Jobseeker's Allowance",
                "es": "Prestación por Desempleo",
                "fr": "Allocation chômage"
              }
            },
            {
              "value": "winter_fuel",
              "text": "Winter Fuel Payment",
              "visibleIf": "{survey.locale} != 'es' && {survey.locale} != 'fr'"
            },
            {
              "value": "housing",
              "text": {
                "default": "Housing Benefit",
                "es": "Ayuda al alquiler",
                "fr": "Aide au logement"
              }
            },
            {
              "value": "pension_credit",
              "text": "Pension Credit",
              "visibleIf": "{survey.locale} empty or {survey.locale} = 'en'"
            }
          ],
          "showOtherItem": true,
          "otherText": {
            "default": "Other (please specify)",
            "es": "Otro (especifique)",
            "fr": "Autre (préciser)"
          }
        }
      ]
    }
  ],
  "showQuestionNumbers": "off",
  "completedHtml": "<h3>Thank you!</h3>"
}
```
## Live Demo
Try the full working example here: [Open in Plunker](https://plnkr.co/edit/G7jYwCiDfxuzB4a6).

## Learn More

* [SurveyJS Localization and Globalization](https://surveyjs.io/form-library/documentation/survey-localization)
* [Expressions](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic)
* [Custom Header Actions](https://surveyjs.io/form-library/examples/modify-titles-of-survey-elements/)