# Custom Question Type: Dynamic Items with Expression-Based Calculations

## Problem
SurveyJS `ItemValue` objects (used in [`choices`](https://surveyjs.io/form-library/documentation/api-reference/questionselectbase#choices) for dropdowns, radio groups, checkboxes, etc.) are normally static. In many real-world cases you need each choice/row to have dynamically calculated values. For example:

* A product/insurance plan where the price is calculated from other survey answers ({age} * 12 + 100)
* Marking certain options as "recommended" based on logic
* Showing live-updating totals or badges inside a comparison table

This example shows how to:

* Extend ItemValue with custom properties and expression support
* Create a custom question which contains a property of an array of these enhanced items

Useful for: product comparison tables, dynamic pricing selectors, insurance quote calculators, etc.

## Solution

1. **Extend `ItemValue`**  
   Create a custom class (e.g. `CustomCalcItem`) that inherits from `ItemValue` and uses `addExpressionProperty()` to define a dynamic property like `calcResult`. This allows each item to have a live-calculated value based on a SurveyJS expression (e.g. `"{age} * 12 + 100"`).

2. **Register the Custom Item Type**  
   Use `Serializer.addClass()` to register your extended `ItemValue` subclass so it can be serialized and edited in the Survey Creator.

3. **Create a Custom Question**  
   Implement a new question type (e.g. `dynamic-calc-table`) that contains an array property of your custom items (via `createItemValues("items")`).

4. **Enable Expression Re-evaluation**  
   Override `runCondition()` in the question model. After calling `super.runCondition()`, iterate over the items and invoke `item.runConditionCore(values)` on each. This ensures that **every item’s expressions are re-executed** whenever survey data changes.

5. **Automatic Updates**  
   As a result, the `calcResult` (or any expression-bound property) on each item is automatically updated in real time — no manual triggering required.

6. **Rendering**  
   In your custom renderer (React, Angular, Vue, or Knockout), simply read:
   - `item.text` → display name
   - `item.calcResult` → computed price, score, etc.
   - `item.isRecommended` → highlight as suggested option  
   …and render them however you like (table, cards, comparison grid, etc.).

### Code Sample

**Important**: This code contains only the question model logic. No Angular/React/Vue renderer component is included. To display the question, implement a custom renderer and register it with your framework of choice.
See official documentation for [Angular](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-angular), [React](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-react), [Vue 3](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-vue).

```js
import { ItemValue, Serializer, Question, ElementFactory } from "survey-core";

// ------------------------------------------------------------------
// 1. Custom ItemValue with expression support
// ------------------------------------------------------------------
const CUSTOM_ITEM_TYPE = "customcalcitem";

export class CustomCalcItem extends ItemValue {
  constructor(value: any = null, text: string = "", typeName: string = CUSTOM_ITEM_TYPE) {
    super(value, text, typeName);

    // Adds a dynamic property which contains a SurveyJS expression
    this.addExpressionProperty("calcResult", (obj: any, result: any) => {
      if (result !== undefined) {
        this.calcResult = result; // store the computed value
      }
    });
  }

  // Result of the expression (updated automatically)
  get calcResult(): any {
    return this.getPropertyValue("calcResult");
  }
  set calcResult(val: any) {
    this.setPropertyValue("calcResult", val);
  }

  // The expression string (e.g. "{age} * 15 + 100")
  get calcExpression(): string {
    return this.getPropertyValueWithoutDefault("calcExpression");
  }
  set calcExpression(val: string) {
    this.setPropertyValue("calcExpression", val);
  }

  // Example extra flag
  get isRecommended(): boolean {
    return !!this.getPropertyValue("isRecommended");
  }
  set isRecommended(val: boolean) {
    this.setPropertyValue("isRecommended", val);
  }
}

// Register the custom ItemValue class
Serializer.addClass(
  CUSTOM_ITEM_TYPE,
  [
    { name: "value", displayName: "Value" },
    { name: "text", displayName: "Display Text" },
    { name: "calcExpression:expression", displayName: "Calculation Expression" },
    { name: "calcResult", isSerializable: false, visible: false },
    { name: "isRecommended:boolean", displayName: "Recommended", default: false }
  ],
  () => new CustomCalcItem(),
  "itemvalue"
);

// ------------------------------------------------------------------
// 2. Custom Question that uses an array of CustomCalcItem
// ------------------------------------------------------------------
const CUSTOM_TABLE_QUESTION_TYPE = "dynamic-calc-table";

export class DynamicCalcTableQuestion extends Question {
  constructor(name: string) {
    super(name);
    this.createItemValues("items"); // creates observable array
  }

  public getType(): string {
    return CUSTOM_TABLE_QUESTION_TYPE;
  }

  // Re-run expressions inside every item when survey values change
  public override runCondition(values: any, properties: any): void {
    super.runCondition(values, properties);
    this.items.forEach(item => item.runConditionCore(values));
  }

  // Public accessor for the items array
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
      "text": "Basic Plan",
      "calcExpression": "{age} < 30 ? 49 : 79",
      "isRecommended": true
    },
    {
      "value": "pro",
      "text": "Pro Plan",
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