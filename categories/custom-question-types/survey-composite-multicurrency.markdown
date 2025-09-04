# Multi-Currency Input for Survey Creator

## Problem
Survey creators need a multi-currency input with custom behavior, allowing them to configure the thousand separator, decimal separator, maximum value, minimum value, mask placeholder, and dropdown choices within a composite question.

## Solution
Register a custom composite question named `"multicurrency"` with a text input for the amount and a dropdown for currency selection. The text input uses a currency mask, and the dropdown allows users to add custom currency choices. Mask settings of an inner Text input field are updated once a survey editor updates corresponding mask settings within the composite element. Survey creators may adjust separators, value limits, placeholders, and currency options.

### Code Sample
```javascript
import { Serializer, ComponentCollection, ItemValue } from "survey-core";

ComponentCollection.Instance.add({
  name: "multicurrency",
  title: "Multi-Currency",
  iconName: "icon-text",
  defaultQuestionTitle: "What is your expected salary and preferred currency?",
  elementsJSON: [
    {
      titleLocation: "hidden",
      title: "Amount",
      name: "amount",
      type: "text",
      inputType: "text",
      maskType: "currency",
      maskSettings: {},
    },
    {
      titleLocation: "hidden",
      title: "Currency",
      name: "currency",
      type: "dropdown",
      inputType: "dropdown",
      startWithNewLine: false,
    },
  ],
  onInit() {
    Serializer.addProperties("multicurrency", [
      {
        name: "maskSettings:masksettings",
        category: "mask",
        visibleIndex: 0,
      },
      {
        name: "currencyChoices",
        category: "currency",
        type: "itemvalues",
        displayName: "Currency",
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
  updateMaskSettings(compositeElement, maskedQuestion) {
    maskedQuestion.maskSettings.fromJSON(compositeElement.maskSettings);
    compositeElement.maskSettings = maskedQuestion.maskSettings;
  },
  updateCurrencyChoices(compositeElement, dropdownQuestion) {
    const currencyChoices = compositeElement.currencyChoices.map((cc) => {
      return new ItemValue(cc.value, cc.text);
    });
    dropdownQuestion.choices = currencyChoices;
  },
  onLoaded(question) {
    let amount = question.contentPanel.getQuestionByName("amount");
    let currency = question.contentPanel.getQuestionByName("currency");
    if (!!amount) {
      this.updateMaskSettings(question, amount);
    }
    if (!!currency) {
      this.updateCurrencyChoices(question, currency);
    }
  },
  onPropertyChanged(question, propertyName, newValue) {
    if (propertyName === "maskSettings") {
      let amount = question.getQuestionByName("amount");
      if (!!amount) {
        this.updateMaskSettings(question, amount);
      }
    }
    if (propertyName === "currencyChoices") {
      let currency = question.getQuestionByName("currency");

      if (!!currency) {
        this.updateCurrencyChoices(question, currency);
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
          "type": "multicurrency",
          "name": "question1",
          "maskSettings": {
            "precision": 0,
            "min": 0,
            "max": 1000000
          },
          "currencyChoices": [
            {
              "value": "USD",
              "text": "US Dollar"
            },
            {
              "value": "EUR",
              "text": "Euro"
            },
            {
              "value": "GBP",
              "text": "British Pound"
            }
          ]
        }
      ]
    }
  ],
  "headerView": "advanced"
}
```

## Learn More
* For more information about available mask types, refer to [Masked Input Fields](https://surveyjs.io/form-library/examples/masked-input-fields/).
* To learn how to register custom properties within composite elements, refer to [Add Custom Properties to Composite Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/create-composite-question-types#add-custom-properties-to-composite-question-types).