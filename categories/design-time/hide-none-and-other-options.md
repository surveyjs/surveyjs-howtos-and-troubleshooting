# Hide the "None" and "Other" Options in Survey Creator

## Problem

I want to prevent survey authors from adding the special "None" and "Other" choices to multiple-choice questions, such as Radio Button Group, Checkboxes, and Dropdown. These options should not appear in either the Property Grid or the design surface.

## Solution

The "None" and "Other" options are controlled by the [`showNoneItem`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#showNoneItem) and [`showOtherItem`](https://surveyjs.io/form-library/documentation/api-reference/dropdown-menu-model#showOtherItem) properties. To hide them, set their `visible` attribute to `false` using the `Serializer` API.

### Code Sample

```javascript
import { Serializer } from "survey-core";

const hiddenProps = [ "showNoneItem", "showOtherItem" ];
hiddenProps.forEach((prop) => {
  Serializer.getProperty("selectbase", prop).visible = false;
});
// ...
// Omitted: `SurveyCreatorModel` creation
// ...
```

## Learn More

[Survey Creator: Property Grid Customization](https://surveyjs.io/survey-creator/documentation/property-grid-customization)