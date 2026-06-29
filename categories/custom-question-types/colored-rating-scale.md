# Create a Colored Numeric Rating Scale

## Problem

I need a numeric rating scale (for example, from 1 to 10) in which each option has its own color. 

## Solution

SurveyJS lets you override how each rating item is rendered via the [`itemComponent`](https://surveyjs.io/form-library/documentation/api-reference/rating-scale-question-model#itemComponent) property. The custom component is registered with `ReactElementFactory` and referenced in the survey JSON:
```json
"itemComponent": "custom-rate-item"
```
The implementation consists of the following steps:

1. Create a custom rating item component that extends `SurveyElementBase` and registers it with `ReactElementFactory`.
3. Import the custom item component in your survey component so the registration runs at startup.
4. Reference the registered component name in the Rating question's `itemComponent` property.
5. Add CSS to size the icons and define hover effects.

### Code Sample

```javascript
// ColoredRatingItem.tsx
import React from "react";
import { Serializer } from "survey-core";
import type { QuestionRatingModel, RenderedRatingItem } from "survey-core";
import { ReactElementFactory, SurveyElementBase } from "survey-react-ui";

Serializer.addProperty("itemvalue", "color");

function hexToRgb(hex: string): string {
  const cleanHex = hex.replace("#", "");
  const bigint = parseInt(cleanHex, 16);
  const r = (bigint >> 16) & 255;
  const g = (bigint >> 8) & 255;
  const b = bigint & 255;
  return `${r}, ${g}, ${b}`;
}

interface ColoredRatingItemProps {
  question: QuestionRatingModel;
  item: RenderedRatingItem & { color?: string };
  index: number;
  handleOnClick: (event: React.MouseEvent<HTMLInputElement>) => void;
  isDisplayMode: boolean;
}

class ColoredRatingItem extends SurveyElementBase<
  ColoredRatingItemProps,
  object
> {
  get question(): QuestionRatingModel {
    return this.props.question;
  }

  get item(): ColoredRatingItemProps["item"] {
    return this.props.item;
  }

  get index(): number {
    return this.props.index;
  }

  handleOnMouseDown(): void {
    this.question.onMouseDown();
  }

  renderElement(): React.JSX.Element {
    const isSelected = this.question.value === this.item.value;
    const mainColor = this.item.color || "#9e9e9e";
    const lightColor = `rgba(${hexToRgb(mainColor)}, 0.2)`;

    return (
      <label
        onMouseDown={() => this.handleOnMouseDown()}
        className={`colored-rating-item__label ${this.question.getItemClass(
          this.item
        )}`}
        onMouseOver={() => this.question.onItemMouseIn(this.item)}
        onMouseOut={() => this.question.onItemMouseOut(this.item)}
        style={{
          ["--item-color" as string]: mainColor,
          ["--item-light" as string]: lightColor,
        }}
      >
        <input
          type="radio"
          className="sv-visuallyhidden"
          name={this.question.questionName}
          id={this.question.getInputId(this.index)}
          value={this.item.value}
          disabled={this.question.isDisabledAttr}
          readOnly={this.question.isReadOnlyAttr}
          checked={isSelected}
          onClick={this.props.handleOnClick}
          onChange={() => {}}
          aria-required={this.question.ariaRequired}
          aria-label={this.item.text || String(this.item.value)}
          aria-invalid={this.question.ariaInvalid}
          aria-errormessage={this.question.ariaErrormessage}
        />
        <div className="colored-rating-item__content">
          <div className="colored-rating-item__text">
            {SurveyElementBase.renderLocString(this.item.locText)}
          </div>
          <span className="colored-rating-item__marker">
            <span className="colored-rating-item__marker-inner" />
          </span>
        </div>
      </label>
    );
  }
}

ReactElementFactory.Instance.registerElement("custom-rate-item", (props) => {
  return React.createElement(ColoredRatingItem, props);
});

export default ColoredRatingItem;
```

### Survey JSON Schema

```json
{
  widthMode: "static",
  width: "1000px",
  showQuestionNumbers: "off",
  pages: [
    {
      name: "page1",
      elements: [
        {
          type: "rating",
          name: "rating-colored",
          title: "How would you rate your experience?",
          itemComponent: "custom-rate-item",
          displayMode: "buttons",
          rateValues: [
            { value: 1, text: "1", color: "#FF0000" },
            { value: 2, text: "2", color: "#FF3300" },
            { value: 3, text: "3", color: "#FF6600" },
            { value: 4, text: "4", color: "#FF9900" },
            { value: 5, text: "5", color: "#FFCC00" },
            { value: 6, text: "6", color: "#FFFF00" },
            { value: 7, text: "7", color: "#99FF00" },
            { value: 8, text: "8", color: "#66FF00" },
            { value: 9, text: "9", color: "#33FF00" },
            { value: 10, text: "10", color: "#00FF00" },
          ],
        },
      ],
    },
  ],
}
```

### Apply Custom Styles

To scope styles to this question, assign a custom CSS class using the [`onUpdateQuestionCssClasses`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUpdateQuestionCssClasses) event:

```js
// ...
// Omitted: `Survey.Model` creation
// ...
survey.onUpdateQuestionCssClasses.add((_sender, options) => {
  if (options.question.name === "rating-colored") {
    options.cssClasses.root += " coloredRadiogroup";
  }
});
```

Add the following styles to your stylesheet:

```css
/* Scoped to the rating-colored question via onUpdateQuestionCssClasses */
.coloredRadiogroup label.colored-rating-item__label.sd-rating__item {
  position: relative;
  display: inline-flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: flex-start;
  box-sizing: border-box;
  width: 56px;
  min-width: 56px;
  height: auto;
  padding: 0;
  margin: 0;
  border: 2px solid transparent;
  border-radius: 12px;
  background-color: var(--item-light) !important;
  color: #161616;
  fill: currentColor;
  box-shadow: none !important;
  white-space: normal;
  font-weight: 600;
  transition: border-color 0.15s ease, transform 0.15s ease;
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--fixed-size {
  width: 56px;
  padding: 0;
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--allowhover:hover {
  background-color: var(--item-light) !important;
  transform: scale(1.04);
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item:focus-within {
  box-shadow: 0 0 0 2px var(--item-color) !important;
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--selected,
.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--selected.sd-rating__item--allowhover:hover {
  background-color: var(--item-light) !important;
  color: #161616 !important;
  border-color: var(--item-color);
  box-shadow: none !important;
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--selected:focus-within {
  box-shadow: 0 0 0 2px var(--item-color) !important;
}

.coloredRadiogroup .colored-rating-item__content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 12px 8px 10px;
}

.coloredRadiogroup .colored-rating-item__text {
  font-size: 16px;
  font-weight: 700;
  color: #161616;
  line-height: 1;
}

.coloredRadiogroup .colored-rating-item__marker {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 2px solid #bdbdbd;
  background-color: transparent;
  flex-shrink: 0;
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--selected .colored-rating-item__marker {
  border-color: var(--item-color);
}

.coloredRadiogroup .colored-rating-item__marker-inner {
  display: none;
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: var(--item-color);
}

.coloredRadiogroup label.colored-rating-item__label.sd-rating__item--selected .colored-rating-item__marker-inner {
  display: block;
}

.coloredRadiogroup fieldset {
  gap: 8px;
}

.strawberry-rating-item__icon-wrapper {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 56px;
  height: 56px;
}

.strawberry-rating-item__icon {
  width: 48px;
  height: 48px;
  transition: opacity 0.15s ease, filter 0.15s ease, transform 0.15s ease;
}

.sd-rating__item--allowhover:hover .strawberry-rating-item__icon {
  transform: scale(1.08);
}
```

## Live Demo

[Open in CodeSandbox](https://codesandbox.io/p/devbox/condescending-roman-wdf6xl)