# Format Date Input in SurveyJS to Match Local Conventions

## Problem
In a SurveyJS JSON configuration, a Text question with `inputType: "date"` uses the `today()` expression to set a default date value. However, the `today()` function always gives it in the format `YYYY-MM-DD` (e.g., `"2025-10-14"`). How to apply a format to match a user's local format.

## Solution
The SurveyJS Text question with `inputType: "date"` uses a standard HTML `<input type="date">` element. The display format of this element is determined by the user's browser and operating system locale settings, while the stored and submitted value is always in the ISO format (`YYYY-MM-DD`). To ensure the date is displayed in the user's local format (e.g., `dd/MM/yyyy` for some regions), the user's device must have the appropriate regional settings configured:

- **Browser Language**: Set to the user's preferred language and region.
- **OS Regional Settings**: Configured to the desired regional format.

When these settings match the user's locale, the `<input type="date">` field will automatically display the date in the corresponding local format.

If you need to enforce a specific display format or provide a custom date picker experience, you can integrate a third-party date picker component into SurveyJS. This allows greater control over the display format and user interaction. Refer to the SurveyJS documentation for integrating third-party components in your framework of choice:

- [React](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-react)
- [Angular](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-angular)
- [Vue](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-vue)

Alternatively, you can use SurveyJS's input mask feature to enforce a specific date format. See the [Date Input Mask Demo](https://surveyjs.io/form-library/examples/masked-input/) for an example.
