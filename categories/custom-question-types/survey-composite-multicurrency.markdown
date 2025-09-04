# Multi-Currency Input for Survey Creator

## Problem
Survey creators need a multi-currency input with custom behavior, allowing them to configure the thousand separator, decimal separator, maximum value, minimum value, mask placeholder, and dropdown choices within a composite question.

## Solution
Register a custom composite question named "multicurrency" with a text input for the amount and a dropdown for currency selection. The text input uses a currency mask, and the dropdown allows customizable choices. Mask settings are dynamically updated based on user changes within the composite element, providing flexibility for survey creators to adjust separators, value limits, placeholders, and currency options.

### Code Sample
```javascript
import { Serializer, ComponentCollection } from "survey-core";

ComponentCollection.Instance.add({
  name: "multicurrency",
  title: "Multi-Currency",
  iconName: "icon-currency",
  elementsJSON: [
    {
      titleLocation: "hidden",
      title: "Amount",
      name: "amount",
      type: "text",
      inputType: "text",
      maskType: "currency",
      maskSettings: {}
    },
    {
      titleLocation: "hidden",
      title: "Currency",
      name: "currency",
      type: "dropdown",
      inputType: "dropdown",
      choices: [],
      startWithNewLine: false,
      minWidth: "150px",
      maxWidth: "150px",
    },
  ],
  onInit() {
    Serializer.addProperties("multicurrency", [
      {
        name: "maskType",
        category: "mask",
        default: "currency",
        visibleIndex: 0,
        choices: ["none", "pattern", "datetime", "numeric", "currency"],
      },
      {
        name: "maskSettings:masksettings",
        category: "mask",
        visibleIndex: 1,
      },
    ]);
  },
  updateMaskSettings(compositeElement, maskedQuestion) {
    maskedQuestion.maskType = compositeElement.maskType;
    maskedQuestion.maskSettings.fromJSON(compositeElement.maskSettings);
    compositeElement.maskType = maskedQuestion.maskType;
    compositeElement.maskSettings = maskedQuestion.maskSettings;
  },
  onLoaded(question) {
    let amount = question.contentPanel.getQuestionByName("amount");
    if (!!amount) {
      this.updateMaskSettings(question, amount);
    }
  },
  onPropertyChanged(question, propertyName, newValue) {
    if (propertyName == "maskType" || propertyName == "maskSettings") {
      let amount = question.contentPanel.getQuestionByName("amount");
      if (!!amount) {
        this.updateMaskSettings(question, amount);
      }
    }
  },
});
```

### Survey JSON Schema
```json
{
  "elements": [
    {
      "type": "multicurrency",
      "name": "multiCurrencyQuestion",
      "title": "Multi-Currency Input",
      "maskType": "currency",
      "maskSettings": {
        "thousandsSeparator": ",",
        "decimalSeparator": ".",
        "min": 0,
        "max": 1000000,
        "placeholder": "Enter amount"
      },
      "amount": {
        "maskType": "currency",
        "maskSettings": {
          "thousandsSeparator": ",",
          "decimalSeparator": ".",
          "min": 0,
          "max": 1000000,
          "placeholder": "Enter amount"
        }
      },
      "currency": {
        "choices": [
          { "value": "USD", "text": "US Dollar" },
          { "value": "EUR", "text": "Euro" },
          { "value": "GBP", "text": "British Pound" }
        ]
      }
    }
  ]
}
```

## Learn More
For additional details on customizing composite questions and mask settings in SurveyJS, refer to the [SurveyJS Documentation](https://surveyjs.io/Documentation/Library).