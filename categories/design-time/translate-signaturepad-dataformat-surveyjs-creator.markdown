# Translate `dataFormat` Property Values for `signaturepad` Question in SurveyJS Creator

## Problem
You want to translate the `dataFormat` property values ("PNG", "JPEG", "SVG") for the `signaturepad` question type in the SurveyJS Creator property grid to display localized names (e.g., in Chinese) instead of their default English names.

## Solution
By default, the `dataFormat` property values ("PNG", "JPEG", "SVG") for the `signaturepad` question in SurveyJS Creator are displayed in their original English names, regardless of the Creator's UI localization. To translate these values, you need to override the default choices by setting lowercase values (`"png"`, `"jpeg"`, `"svg"`) using `Survey.Serializer.findProperty().setChoices()`. This removes the predefined uppercase texts, allowing you to define custom localized names in the target locale's `pv` (property values) object.

The following code example demonstrates how to translate the `dataFormat` property values into Chinese (Simplified) for the `signaturepad` question in SurveyJS Creator.

## Code Example

```javascript
// Override default dataFormat property choices to enable localization
Survey.Serializer.findProperty("signaturepad", "dataFormat").setChoices(["png", "jpeg", "svg"]);

// Get the Chinese (Simplified) locale strings
const chLocale = SurveyCreator.getLocaleStrings("zh-cn");

// Define translated names for dataFormat property values
chLocale.pv.png = "CH-PNG";
chLocale.pv.jpeg = "CH-JPEG";
chLocale.pv.svg = "CH-SVG";

// Initialize SurveyJS Creator and set the locale
const creator = new SurveyCreator.SurveyCreator({
  showTranslationTab: true
});
creator.locale = "zh-cn";

// Render the creator
creator.render("surveyCreatorContainer");
```

### Notes
- **Dependencies**: Ensure you have included `survey-core` and `survey-creator-core` with their CSS files:
  ```bash
  npm install survey-core survey-creator-core
  ```
- **Overriding Default Choices**: The `Survey.Serializer.findProperty("signaturepad", "dataFormat").setChoices(["png", "jpeg", "svg"])` call replaces the default uppercase "PNG", "JPEG", and "SVG" values with lowercase equivalents, enabling custom localization.
- **Localization**: The `pv` object in the locale strings is used to assign translated names (e.g., "CH-PNG", "CH-JPEG", "CH-SVG") for the `dataFormat` property values in the Chinese (Simplified) locale (`zh-cn`).
- **Other Languages**: Replace `"zh-cn"` with the desired locale code (e.g., `"fr"` for French) and update the translated strings (e.g., `chLocale.pv.png = "PNG-Français"`).
- **Plunker Reference**: For a related example, see the [Plunker example](https://plnkr.co/edit/EdAKE6J0Svjxr1Ty).
- **Customization**: Extend this approach to translate other property values by modifying the `pv` object for additional properties or question types. Ensure the `setChoices` method is called before setting localized strings to avoid conflicts with default values.