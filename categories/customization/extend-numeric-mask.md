# Year of Birth Input with Numeric Mask (No Thousand Separator, Custom Placeholder)

## Problem

How to create a **year of birth** input field in SurveyJS with the following requirements:

- Use a numeric input mask
- Allow only 4-digit years (e.g., 1925–2025)
- Show a meaningful **placeholder** like "Enter a Year" instead of underscores (____)
- Prevent thousand separators (no 1,234 formatting)
- Restrict input to realistic birth years

The default `pattern` mask in SurveyJS inserts underscores for unfilled digits. However, the `numeric` mask applies thousand separators. These options are both undesirable for a year field.

## Solution

Extend the built-in `InputMaskNumeric` class, override the masking behavior to:

- Disable thousand separators
- Set precision to 0 (whole numbers only)
- Define min/max year range
- Register the new mask type so it can be used via `"maskType": "year"`

### Code Sample

```javascript
class YearMask extends Survey.InputMaskNumeric {
  constructor() {
    super();
    this.allowNegativeValues = false;
    this.precision = 0;           // no decimal places
    this.min = 1900;              // adjust min year as needed
    this.max = new Date().getFullYear(); // current year
  }

  getType() {
    return "yearmask";
  }

  getNumberMaskedValue(src, matchWholeMask = false) {
    const parsedNumber = super.parseNumber(src);
        
    if (!super.validateNumber(parsedNumber, matchWholeMask)) {
      return null;
    }

    // Display without thousand separators
    return this.displayNumber(parsedNumber, false, matchWholeMask);
  }
}

// Register the custom mask type
Survey.Serializer.addClass(
  "yearmask",
  [],
  function () {
    return new YearMask();
  },
  "numericmask"   // inherit from numericmask
);
```

### Question JSON Schema

```
{
   "type": "text",
   "name": "year-of-birth",
   "title": "Year of birth",
   "maskType": "year",
   "placeholder": "Enter a Year"          
}
```

### Live Demo

[View in Plunker](https://plnkr.co/edit/yJRhf4J5u2A4JI8N)