# Dynamic Choice Options with Expression-Based Calculations

## Problem

I need each choice item in a Dropdown, Radio Button Group, or Checkboxes question to have a value that is calculated dynamically based on survey data. For example, a plan selector might compute price from `{age}` or other answers.

## Solution

SurveyJS choice items (the `ItemValue` class) support only a fixed set of properties, so expression-based values must be added through customization. Extend the `ItemValue` class and use it inside a custom question type. The steps below show how to introduce an expression property that automatically recalculates whenever survey data changes.

1. **Extend `ItemValue`**  
   Create a subclass (e.g. `CustomCalcItem`) that calls `addExpressionProperty()` to define a dynamic property like `calcResult`. This allows each item to have a live-calculated value based on a SurveyJS expression (e.g. `"{age} * 12 + 100"`).

2. **Register the custom item type**  
   Use `Serializer.addClass()` to register your `CustomCalcItem` subclass, so it can be recognized and serialized by SurveyJS and edited in Survey Creator.

3. **Create a custom question**  
   Implement a new question type (e.g., `dynamic-calc-table`) that contains an array of your custom items using `createItemValues("items")`.

4. **Recalculate expressions automatically**  
   Override `runCondition()` in your custom question type. After calling `super.runCondition()`, iterate over items and call `runConditionCore(values)` on each item. This ensures that every item's expression is re-evaluated whenever the respondent updates answers, and the result is saved in `calcResult` (or any other expression-bound property) in real time&mdash;no manual re-evaluation needed.

5. **Render the dynamic values**  
   In your custom renderer (React, Angular, or Vue 3), use `item.text` for display text and `item.calcResult` for the computed value (price, score, weight, etc.). Render the items in whatever layout you need: table, grid, cards, comparison blocks, and so on.

### Code Sample

> The snippet below contains only the question model implementation. To display the question, implement a renderer for your chosen framework using these instructions:
>
> - [Angular](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-angular#render-the-third-party-component)
> - [React](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-react#render-the-third-party-component)
> - [Vue 3](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-vue#render-the-third-party-component)

```js
import { ItemValue, Serializer, Question, ElementFactory } from "survey-core";

// ------------------------------------------------------------------
// 1. Custom ItemValue with expression support
// ------------------------------------------------------------------
const CUSTOM_ITEM_TYPE = "customcalcitem";

export class CustomCalcItem extends ItemValue {
  constructor(value: any = null, text: string = "", typeName: string = CUSTOM_ITEM_TYPE) {
    super(value, text, typeName);

    // Adds a dynamic property bound to a SurveyJS expression
    this.addExpressionProperty("calcResult", (obj: any, result: any) => {
      if (result !== undefined) {
        this.calcResult = result;
      }
    });
  }

  // Runtime-calculated result
  get calcResult(): any {
    return this.getPropertyValue("calcResult");
  }
  set calcResult(val: any) {
    this.setPropertyValue("calcResult", val);
  }

  // The expression (e.g. "{age} * 15 + 100")
  get calcExpression(): string {
    return this.getPropertyValueWithoutDefault("calcExpression");
  }
  set calcExpression(val: string) {
    this.setPropertyValue("calcExpression", val);
  }
}

// Register the custom ItemValue class
Serializer.addClass(
  CUSTOM_ITEM_TYPE,
  [
    { name: "value", displayName: "Value" },
    { name: "text", displayName: "Display Text" },
    { name: "calcExpression:expression", displayName: "Calculation expression" },
    { name: "calcResult", isSerializable: false, visible: false }
  ],
  () => new CustomCalcItem(),
  "itemvalue"
);

// ------------------------------------------------------------------
// 2. Custom Question that holds an array of CustomCalcItem
// ------------------------------------------------------------------
const CUSTOM_TABLE_QUESTION_TYPE = "dynamic-calc-table";

export class DynamicCalcTableQuestion extends Question {
  constructor(name: string) {
    super(name);
    this.createItemValues("items");
  }

  public getType(): string {
    return CUSTOM_TABLE_QUESTION_TYPE;
  }

   // Re-run each item's expression when survey values change
  public override runCondition(values: any, properties: any): void {
    super.runCondition(values, properties);
    this.items.forEach(item => item.runConditionCore(values));
  }

  public get items(): Array<CustomCalcItem> {
    return this.getPropertyValue("items");
  }
  public set items(val: Array<CustomCalcItem>) {
    this.setPropertyValue("items", val);
  }
}

// Register the question type
ElementFactory.Instance.registerElement(
  CUSTOM_TABLE_QUESTION_TYPE,
  (name: string) => new DynamicCalcTableQuestion(name)
);

Serializer.addClass(
  CUSTOM_TABLE_QUESTION_TYPE,
  [
    {
      name: "items",
      className: CUSTOM_ITEM_TYPE,
      isArray: true,
      displayName: "Items",
      category: "general"
    }
  ],
  () => new DynamicCalcTableQuestion(""),
  "question"
);
```

### Survey JSON Schema

```json
{
  "name": "planSelector",
  "type": "dynamic-calc-table",
  "title": "Available Plans",
  "items": [
    {
      "value": "basic",
      "text": "Basic",
      "calcExpression": "{age} < 30 ? 49 : 79"
    },
    {
      "value": "pro",
      "text": "Pro",
      "calcExpression": "{age} * 3 + 50"
    },
    {
      "value": "enterprise",
      "text": "Enterprise",
      "calcExpression": "250"
    }
  ]
}
```

## Learn More

- [Conditional Logic and Dynamic Texts](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic)
- [Add Custom Properties to Question Types](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form)