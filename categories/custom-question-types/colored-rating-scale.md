# Create a Colored Numeric Rating Scale

## Problem

I need a numeric rating scale (for example, from 1 to 10) in which each option has its own color. The built-in SurveyJS [Rating Scale](https://surveyjs.io/form-library/examples/rating-scale/) question does not support per-item color customization, and the Smileys variation is limited to a predefined color set. This makes it unsuitable when you require full control over individual option styling.

## Solution

Use a [Radio Button Group](https://surveyjs.io/form-library/examples/single-select-radio-button-group/) question with a custom item component that renders each option as a colored element.

The implementation consists of the following steps:

1. Register a custom `color` property for choice options (`"itemvalue"`) using the [`Serializer` API](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form#add-custom-properties-to-an-existing-class).
2. Create a custom component that renders each radio item as a colored element displaying its value.
3. Assign this custom component to the question via the [`itemComponent`](https://surveyjs.io/form-library/documentation/api-reference/radio-button-question-model#itemComponent) property.
4. Specify colors for each choice in the `choices` array.

### Code Sample

```javascript
// Converts HEX color to RGB string (used for rgba values)
function hexToRgb(hex) {
  const cleanHex = hex.replace("#", "");
  const bigint = parseInt(cleanHex, 16);
  const r = (bigint >> 16) & 255;
  const g = (bigint >> 8) & 255;
  const b = bigint & 255;
  return `${r}, ${g}, ${b}`;
}

// 1. Register custom `color` property on choice options
Survey.Serializer.addProperty("itemvalue", "color");

// 2. Create a component for rendering custom radio items
class CustomRadioItem extends SurveyReact.SurveyQuestionRadioItem {
  get item() {
    return this.props.item;
  }

  renderElementContent() {
    const question = this.question;
    const item = this.item;
    const isSelected = question.isItemSelected(item);

    const mainColor = item.color || "#9e9e9e";
    const lightColor = `rgba(${hexToRgb(mainColor)}, 0.12)`;
    const selectedColor = `rgba(${hexToRgb(mainColor)}, 0.25)`;
    const borderColor = isSelected ? mainColor : "transparent";

    const handleOnChange = (event) => {
      question.clickItemHandler(item, event.currentTarget.checked);
    };

    const handleOnMouseDown = () => {
      question.onMouseDown();
    };

    return (
      <div role="presentation" className="item-root">
        <label
          className={`item-label ${isSelected ? "item-label--selected" : ""}`}
          onMouseDown={handleOnMouseDown}
          style={{
            "--item-color": mainColor,
            "--item-light": lightColor,
            "--item-selected": selectedColor,
            "--item-border": borderColor,
          }}
        >
          <input
            role="option"
            className="item-input"
            id={question.getItemId(item)}
            type="radio"
            name={question.questionName}
            checked={this.isChecked}
            value={item.value}
            onChange={handleOnChange}
          />
          <div className="item-content">
            <div className="item-text">{item.text || item.value}</div>
            <span className="radio-outer">
              <span className="radio-inner"></span>
            </span>
          </div>
        </label>
      </div>
    );
  }
}

SurveyReact.ReactElementFactory.Instance.registerElement(
  "custom-radio-item",
  (props) => React.createElement(CustomRadioItem, props)
);
```

### Survey JSON Schema

```json
{
  "widthMode": "static",
  "width": "1000px",
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "radiogroup",
          "name": "rating-colored",
          "title": "How would you rate your experience?",
          // 3. Assign the custom component to the question
          "itemComponent": "custom-radio-item",
          "colCount": 0,
          // 4. Specify colors for each choice
          "choices": [
            { "value": 1, "text": "1", "color": "#FF0000" },
            { "value": 2, "text": "2", "color": "#FF3300" },
            { "value": 3, "text": "3", "color": "#FF6600" },
            { "value": 4, "text": "4", "color": "#FF9900" },
            { "value": 5, "text": "5", "color": "#FFCC00" },
            { "value": 6, "text": "6", "color": "#FFFF00" },
            { "value": 7, "text": "7", "color": "#99FF00" },
            { "value": 8, "text": "8", "color": "#66FF00" },
            { "value": 9, "text": "9", "color": "#33FF00" },
            { "value": 10, "text": "10", "color": "#00FF00" }
          ]
        }
      ]
    }
  ]
}
```

### Apply Custom Styles

To scope styles to this question, assign a custom CSS class using the [`onUpdateQuestionCssClasses`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUpdateQuestionCssClasses) event:

```js
// ...
// Omitted: `Survey.Model` creation
// ...
survey.onUpdateQuestionCssClasses.add((_, options) => {
  if (options.question.name === "rating-colored") {
    options.cssClasses.root += " coloredRadiogroup";
  }
});
```

Add the following styles to your stylesheet:

```css
.coloredRadiogroup .item-root {
  display: inline-block;
}

.coloredRadiogroup .item-label {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-width: 54px;
  width: 54px;
  padding: 10px 4px;
  border-radius: 10px;
  background: var(--item-light, rgba(158, 158, 158, 0.12));
  border: 2px solid var(--item-border, transparent);
  cursor: pointer;
  transition: all 0.18s ease;
  user-select: none;
}

.coloredRadiogroup .item-label:hover {
  background: rgba(0, 0, 0, 0.06);
  transform: translateY(-2px);
}

.coloredRadiogroup .item-label.item-label--selected {
  background: var(--item-selected, rgba(25, 179, 148, 0.25));
  border: 2px solid var(--item-color);
  box-shadow: 0 0 0 4px rgba(var(--item-color-rgb, 25, 179, 148), 0.18);
  transform: scale(1.06);
}

.coloredRadiogroup .item-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.coloredRadiogroup .item-text {
  font-size: 18px;
  min-height: 26px;
  font-weight: 700;
  color: #1a1a1a;
}

.coloredRadiogroup .radio-outer {
  width: 18px;
  height: 18px;
  border: 2px solid #888;
  border-radius: 50%;
  background: white;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}

.coloredRadiogroup .item-label:hover .radio-outer {
  border-color: #555;
}

.coloredRadiogroup .item-label.item-label--selected .radio-outer {
  border-color: var(--item-color);
  background: var(--item-color);
}

.coloredRadiogroup .radio-inner {
  display: none;
  width: 8px;
  height: 8px;
  background: white;
  border-radius: 50%;
}

.coloredRadiogroup .item-label.item-label--selected .radio-inner {
  display: block;
}

.coloredRadiogroup .item-input {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

.coloredRadiogroup .sd-selectbase__list,
.sd-selectbase--row {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  gap: 12px;
  overflow-x: auto;
  padding: 12px 16px;
  justify-content: flex-start;
  -webkit-overflow-scrolling: touch;
  scroll-padding-left: 16px;
  scroll-snap-type: x proximity;
  margin-right: auto;
  text-align: left;
}

.coloredRadiogroup .sd-question .sd-selectbase__item,
.sd-selectbase--row .sd-selectbase__item {
  flex: 0 0 auto;
}

.coloredRadiogroup .sd-question .sd-selectbase__list::-webkit-scrollbar,
.sd-selectbase--row::-webkit-scrollbar {
  height: 6px;
}

.coloredRadiogroup .sd-question .sd-selectbase__list::-webkit-scrollbar-thumb,
.sd-selectbase--row::-webkit-scrollbar-thumb {
  background: #aaa;
  border-radius: 3px;
}

.coloredRadiogroup .sd-question .sd-selectbase__list::-webkit-scrollbar-track,
.sd-selectbase--row::-webkit-scrollbar-track {
  background: #f1f1f1;
}

@media (max-width: 600px) {
  .coloredRadiogroup .sd-question .sd-selectbase__list,
  .sd-selectbase--row {
    flex-direction: column;
    gap: 8px;
    overflow-x: hidden;
    align-items: flex-start;
    padding: 8px 10px;
    justify-content: flex-start;
  }

  .coloredRadiogroup .sd-selectbase__item {
    width: 100%;
    max-width: 220px;
  }

  .coloredRadiogroup .item-label {
    width: 100%;
    min-width: unset;
    max-width: 220px;
    padding: 8px 10px;
    justify-content: flex-start;
    text-align: left;
  }

  .coloredRadiogroup .item-text {
    font-size: 18px;
  }
}

```

## Live Demo

[Open in CodeSandbox](https://codesandbox.io/p/devbox/vigilant-river-32d3mq)