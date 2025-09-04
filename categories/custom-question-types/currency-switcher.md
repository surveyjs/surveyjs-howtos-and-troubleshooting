# Create a Reusable Currency Switcher in SurveyJS

## Problem

I want to create a [composite question](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types) that combines a numeric input field with a currency dropdown. Survey authors should be able to configure mask settings for the input field (thousands and decimal separators, min/max values, mask placeholder) and customize the list of available currencies.

## Solution

You can implement a reusable composite question with a currency switcher as follows:

1. Register a custom composite question using the `ComponentCollection`.
2. Configure nested fields&mdash;an `amount` numeric input and a `currency` dropdown&mdash;in the [`elementsJSON`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#elementsJSON) array.
3. Add custom properties (`maskSettings` and `currencies`) to allow configuring the mask settings and currency list. These properties can be overridden in the survey JSON schema or Property Grid. Use the [`onInit`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onInit) callback to add these properties.
4. Apply the custom properties initially using the [`onLoaded`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onLoaded) callback.
5. Use the [`onPropertyChanged`](https://surveyjs.io/form-library/documentation/api-reference/icustomquestiontypeconfiguration#onPropertyChanged) callback to update the [`maskSettings`](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model#maskSettings) and [`choices`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#choices) properties of the nested fields dynamically when custom properties change.
6. Add the composite question to your survey JSON schema and configure the custom properties as needed.

### Code Sample

```javascript
import { Serializer, ComponentCollection, ItemValue } from "survey-core";

// Step 1: Register a custom composite question
ComponentCollection.Instance.add({
  name: "multicurrency",
  title: "Multi-Currency",
  iconName: "icon-text",
  defaultQuestionTitle: "What is your expected salary?",
  // Step 2: Configure nested fields
  elementsJSON: [
    {
      name: "amount",
      type: "text",
      titleLocation: "hidden",
      inputType: "text",
      maskType: "currency",
      maskSettings: {},
    },
    {
      name: "currency",
      type: "dropdown",
      titleLocation: "hidden",
      placeholder: "Select currency...",
      startWithNewLine: false,
    },
  ],
  onInit() {
    // Step 3: Add custom properties for mask settings and currency list
    Serializer.addProperties("multicurrency", [
      {
        name: "maskSettings",
        type: "masksettings",
        category: "mask",
        visibleIndex: 0,
      },
      {
        name: "currencies",
        category: "Currencies",
        type: "itemvalues",
        displayName: "Currencies",
        visibleIndex: 1,
        default: [
          { value: "USD", text: "US Dollar" },
          { value: "EUR", text: "Euro" },
          { value: "GBP", text: "British Pound" },
          { value: "JPY", text: "Japanese Yen" },
          { value: "CAD", text: "Canadian Dollar" },
        ],
      },
    ]);
  },
  updateMaskSettings(compositeEl, maskedQ) {
    maskedQ.maskSettings.fromJSON(compositeEl.maskSettings);
    compositeEl.maskSettings = maskedQ.maskSettings;
  },
  updateCurrencies(compositeEl, dropdownQ) {
    const currencies = compositeEl.currencies.map(
      (cc) => new ItemValue(cc.value, cc.text);
    );
    dropdownQ.choices = currencies;
  },
  // Step 4: Apply the custom properties initially
  onLoaded(question) {
    const amount = question.contentPanel.getQuestionByName("amount");
    const currency = question.contentPanel.getQuestionByName("currency");
    if (amount) {
      this.updateMaskSettings(question, amount);
    }
    if (currency) {
      this.updateCurrencies(question, currency);
    }
  },
  // Step 5: Update nested fields when custom properties change
  onPropertyChanged(question, propertyName) {
    if (propertyName === "maskSettings") {
      const amount = question.getQuestionByName("amount");
      if (amount) {
        this.updateMaskSettings(question, amount);
      }
    }
    if (propertyName === "currencies") {
      const currency = question.getQuestionByName("currency");

      if (currency) {
        this.updateCurrencies(question, currency);
      }
    }
  },
});
```

### Survey JSON Schema

```json
{
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "name": "question1",
          "type": "multicurrency",
          "title": "Please specify your desired salary.",
          "maskSettings": {
            "precision": 0,
            "min": 0,
            "max": 1000000
          },
          "currencies": [
            { "value": "USD", "text": "US Dollar" },
            { "value": "EUR", "text": "Euro" },
            { "value": "GBP", "text": "British Pound" }
          ]
        }
      ]
    }
  ]
}
```

## Learn More

- [Add Custom Properties to Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types#add-custom-properties-to-composite-question-types)
- [Masked Input Fields Demo](https://surveyjs.io/form-library/examples/masked-input-fields/)