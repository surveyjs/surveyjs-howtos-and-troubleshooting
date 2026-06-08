# Override the Rating Scale Item Component to Display Custom Images

## Problem

I need a [Rating Scale](https://surveyjs.io/form-library/examples/rating-scale/) question in which each scale item renders a custom image instead of the default stars, numbers, or smileys. The visual intensity should vary by value—for example, on a 1–5 scale, the first item should appear faded and the fifth item should be the brightest, with intermediate values interpolated smoothly.


## Solution

SurveyJS lets you override how each rating item is rendered via the [`itemComponent`](https://surveyjs.io/form-library/documentation/api-reference/rating-scale-question-model#itemComponent) property. The custom component is registered with `ReactElementFactory` and referenced in the survey JSON:

```json
"itemComponent": "strawberry-rating-item"
```

Each item gets a different visual intensity based on its value:

- **1** — faded (`opacity: 0.2`, `brightness: 0.45`)
- **5** — brightest (`opacity: 1.0`, `brightness: 1.3`)

Values in between are interpolated linearly.

The implementation consists of the following steps:

1. Create a custom SVG icon component for the rating items.
2. Create a custom rating item component that extends `SurveyElementBase`, renders the icon with intensity-based styles, and registers it with `ReactElementFactory`.
3. Import the custom item component in your survey component so the registration runs at startup.
4. Reference the registered component name in the Rating question's `itemComponent` property.
5. Add CSS to size the icons and define hover effects.

### Code Sample

**StrawberryIcon.tsx**

```tsx
import { useId } from 'react';
import type { CSSProperties, SVGProps } from 'react';

interface StrawberryIconProps extends SVGProps<SVGSVGElement> {
  style?: CSSProperties;
}

const SEEDS: Array<[number, number, number, number]> = [
  [22, 24, 1.6, 2.2], [28, 22, 1.5, 2.1], [34, 23, 1.6, 2.2], [40, 25, 1.5, 2.0],
  [19, 30, 1.5, 2.1], [25, 29, 1.6, 2.2], [31, 28, 1.5, 2.1], [37, 30, 1.6, 2.2], [43, 32, 1.4, 2.0],
  [17, 36, 1.5, 2.1], [23, 35, 1.6, 2.2], [29, 34, 1.5, 2.1], [35, 35, 1.6, 2.2], [41, 37, 1.5, 2.0], [46, 39, 1.4, 1.9],
  [20, 42, 1.5, 2.1], [26, 41, 1.6, 2.2], [32, 40, 1.5, 2.1], [38, 42, 1.5, 2.0], [44, 44, 1.4, 1.9],
  [24, 47, 1.5, 2.0], [30, 46, 1.5, 2.1], [36, 47, 1.5, 2.0], [42, 49, 1.4, 1.9],
  [28, 52, 1.4, 1.9], [34, 53, 1.4, 1.9], [40, 54, 1.3, 1.8],
  [31, 57, 1.3, 1.8], [37, 58, 1.3, 1.8],
];

export default function StrawberryIcon({ style, className, ...props }: StrawberryIconProps) {
  const uid = useId().replace(/:/g, '');

  return (
    <svg
      viewBox="0 0 64 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      className={className}
      style={style}
      aria-hidden="true"
      {...props}
    >
      <defs>
        <radialGradient id={`${uid}-body`} cx="38%" cy="32%" r="68%">
          <stop offset="0%" stopColor="#FF5A62" />
          <stop offset="45%" stopColor="#E8242F" />
          <stop offset="100%" stopColor="#A80F18" />
        </radialGradient>
        <radialGradient id={`${uid}-shine`} cx="28%" cy="22%" r="32%">
          <stop offset="0%" stopColor="#FFFFFF" stopOpacity="0.95" />
          <stop offset="35%" stopColor="#FFD5DA" stopOpacity="0.55" />
          <stop offset="100%" stopColor="#FFFFFF" stopOpacity="0" />
        </radialGradient>
        <linearGradient id={`${uid}-leaf`} x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" stopColor="#8FD44A" />
          <stop offset="55%" stopColor="#3FAE3A" />
          <stop offset="100%" stopColor="#1F7A24" />
        </linearGradient>
        <linearGradient id={`${uid}-seed`} x1="0%" y1="0%" x2="100%" y2="100%">
          <stop offset="0%" stopColor="#FFF1B8" />
          <stop offset="100%" stopColor="#D4B35A" />
        </linearGradient>
        <clipPath id={`${uid}-clip`}>
          <path d="M32 11c-1.5 0-3.2 0.8-4.4 2.4l-2.8 4.2C14.5 20.5 10 28.5 10 37.5c0 12.5 9.5 22.5 18.5 26.5 9-4 18.5-14 18.5-26.5 0-9-4.5-17-14.8-19.9L37.4 13.4C36.2 11.8 34.5 11 32 11z" />
        </clipPath>
      </defs>

      <g transform="rotate(6 32 36)">
        <path
          d="M32 11c-1.5 0-3.2 0.8-4.4 2.4l-2.8 4.2C14.5 20.5 10 28.5 10 37.5c0 12.5 9.5 22.5 18.5 26.5 9-4 18.5-14 18.5-26.5 0-9-4.5-17-14.8-19.9L37.4 13.4C36.2 11.8 34.5 11 32 11z"
          fill={`url(#${uid}-body)`}
        />

        <g clipPath={`url(#${uid}-clip)`}>
          {SEEDS.map(([cx, cy, rx, ry], index) => (
            <ellipse
              key={index}
              cx={cx}
              cy={cy}
              rx={rx}
              ry={ry}
              fill={`url(#${uid}-seed)`}
              transform={`rotate(-12 ${cx} ${cy})`}
            />
          ))}
        </g>

        <path
          d="M32 11c-1.5 0-3.2 0.8-4.4 2.4l-2.8 4.2C14.5 20.5 10 28.5 10 37.5c0 12.5 9.5 22.5 18.5 26.5 9-4 18.5-14 18.5-26.5 0-9-4.5-17-14.8-19.9L37.4 13.4C36.2 11.8 34.5 11 32 11z"
          fill={`url(#${uid}-shine)`}
        />

        <path
          d="M44 24c6 8 8 16 6 24"
          stroke="#FFFFFF"
          strokeOpacity="0.18"
          strokeWidth="2.2"
          strokeLinecap="round"
        />

        <g fill={`url(#${uid}-leaf)`}>
          <path d="M32 4c-1 0-2 0.4-2.6 1.2l-1.2 1.8c-0.4 0.6-0.2 1.4 0.4 1.8l2.2 1.6c0.5 0.4 1.2 0.2 1.5-0.3l1.4-2.4c0.3-0.5 0.1-1.1-0.4-1.4L33.6 4.6C33.2 4.2 32.6 4 32 4z" />
          <path d="M24 7c-1.8 0.8-3 2.4-3.2 4.2l-0.4 2.8c-0.1 0.7 0.4 1.3 1.1 1.4l2.6 0.3c0.7 0.1 1.3-0.4 1.4-1.1l0.3-2.8c0.1-1.2 0.8-2.3 1.8-3L27.8 7.8C26.8 7.2 25.4 6.8 24 7z" />
          <path d="M40 7c1.8 0.8 3 2.4 3.2 4.2l0.4 2.8c0.1 0.7-0.4 1.3-1.1 1.4l-2.6 0.3c-0.7 0.1-1.3-0.4-1.4-1.1l-0.3-2.8c-0.1-1.2-0.8-2.3-1.8-3L36.2 7.8C37.2 7.2 38.6 6.8 40 7z" />
          <path d="M18 12c-2.2 1.2-3.4 3.4-3.2 5.7l0.5 3.1c0.1 0.7 0.7 1.2 1.4 1.1l2.7-0.4c0.7-0.1 1.2-0.7 1.1-1.4l-0.5-3.1c-0.2-1.4 0.3-2.8 1.3-3.8L21.8 12.8C20.6 12.1 19.2 11.8 18 12z" />
          <path d="M46 12c2.2 1.2 3.4 3.4 3.2 5.7l-0.5 3.1c-0.1 0.7-0.7 1.2-1.4 1.1l-2.7-0.4c-0.7-0.1-1.2-0.7-1.1-1.4l0.5-3.1c0.2-1.4-0.3-2.8-1.3-3.8L42.2 12.8C43.4 12.1 44.8 11.8 46 12z" />
        </g>

        <g stroke="#1F6B22" strokeWidth="0.6" strokeLinecap="round" opacity="0.75">
          <path d="M32 5v6M24 9l3 5M40 9l-3 5M19 14l4 4M45 14l-4 4" />
        </g>
      </g>
    </svg>
  );
}
```

**StrawberryRatingItem.tsx**

```tsx
import React from 'react';
import { ReactElementFactory, SurveyElementBase } from 'survey-react-ui';
import type { QuestionRatingModel, RenderedRatingItem } from 'survey-core';
import StrawberryIcon from './StrawberryIcon';

function getStrawberryStyle(value: number): React.CSSProperties {
  const intensity = (value - 1) / 4;
  return {
    opacity: 0.2 + intensity * 0.8,
    filter: `brightness(${0.45 + intensity * 0.85})`,
  };
}

interface StrawberryRatingItemProps {
  question: QuestionRatingModel;
  item: RenderedRatingItem;
  index: number;
  handleOnClick: (event: React.MouseEvent<HTMLInputElement>) => void;
  isDisplayMode: boolean;
}

class StrawberryRatingItem extends SurveyElementBase<StrawberryRatingItemProps, object> {
  get question(): QuestionRatingModel {
    return this.props.question;
  }

  get item(): RenderedRatingItem {
    return this.props.item;
  }

  get index(): number {
    return this.props.index;
  }

  handleOnMouseDown(): void {
    this.question.onMouseDown();
  }

  renderElement(): React.JSX.Element {
    const value = Number(this.item.value);
    const imageStyle = getStrawberryStyle(value);

    return (
      <label
        onMouseDown={() => this.handleOnMouseDown()}
        className={this.question.getItemClass(this.item.itemValue)}
        onMouseOver={() => this.question.onItemMouseIn(this.item)}
        onMouseOut={() => this.question.onItemMouseOut(this.item)}
      >
        <input
          type="radio"
          className="sv-visuallyhidden"
          name={this.question.questionName}
          id={this.question.getInputId(this.index)}
          value={this.item.value}
          disabled={this.question.isDisabledAttr}
          readOnly={this.question.isReadOnlyAttr}
          checked={this.question.value === this.item.value}
          onClick={this.props.handleOnClick}
          onChange={() => {}}
          aria-required={this.question.ariaRequired}
          aria-label={this.item.text || String(this.item.value)}
          aria-invalid={this.question.ariaInvalid}
          aria-errormessage={this.question.ariaErrormessage}
        />
        <span className="strawberry-rating-item__icon-wrapper" title={this.item.text}>
          <StrawberryIcon
            className="strawberry-rating-item__icon"
            style={imageStyle}
          />
        </span>
      </label>
    );
  }
}

ReactElementFactory.Instance.registerElement('strawberry-rating-item', (props) => {
  return React.createElement(StrawberryRatingItem, props);
});

export default StrawberryRatingItem;
```

**Survey.tsx**

```tsx
'use client'

import { useCallback } from 'react';
import 'survey-core/survey-core.css';
import { Model } from 'survey-core'
import { Survey } from 'survey-react-ui'
import './StrawberryRatingItem';

const surveyJson = {
  elements: [{
    type: "rating",
    name: "strawberryRating",
    title: "How much do you love strawberries?",
    description: "Select a strawberry — the first one is faded, the fifth is the brightest.",
    displayMode: "buttons",
    autoGenerate: false,
    rateValues: [
      { value: 1, text: "1" },
      { value: 2, text: "2" },
      { value: 3, text: "3" },
      { value: 4, text: "4" },
      { value: 5, text: "5" },
    ],
    itemComponent: "strawberry-rating-item",
    minRateDescription: "Not at all",
    maxRateDescription: "Absolutely!",
  }]
};

export default function SurveyComponent() {
  const survey = new Model(surveyJson);
  const alertResults = useCallback((sender: Model) => {
    const results = JSON.stringify(sender.data);
    alert(results);
  }, []);

  survey.onComplete.add(alertResults);

  return (
    <Survey model={survey} />
  );
}
```

### Survey JSON Schema

```json
{
  "elements": [
    {
      "type": "rating",
      "name": "strawberryRating",
      "title": "How much do you love strawberries?",
      "description": "Select a strawberry — the first one is faded, the fifth is the brightest.",
      "displayMode": "buttons",
      "autoGenerate": false,
      "rateValues": [
        { "value": 1, "text": "1" },
        { "value": 2, "text": "2" },
        { "value": 3, "text": "3" },
        { "value": 4, "text": "4" },
        { "value": 5, "text": "5" }
      ],
      "itemComponent": "strawberry-rating-item",
      "minRateDescription": "Not at all",
      "maxRateDescription": "Absolutely!"
    }
  ]
}
```

### Apply Custom Styles

Add the following styles to your stylesheet to size the icons and add a hover scale effect:

```css
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

## Learn More

- [Single- and Multi-Select Questions with Custom Item Template](https://surveyjs.io/form-library/examples/add-custom-items-to-single-and-multi-select-questions/)
- [Example: Custom Question Renderer](https://surveyjs.io/form-library/examples/create-custom-question-renderer/)
