# Format Date Input in SurveyJS to Match Local Conventions

## Problem

In a SurveyJS form, a Single-Line Input question with `inputType: "date"` may display dates in a format that doesn't match the user's local date conventions.

## Solution

The SurveyJS Single-Line Input question with `inputType: "date"` is rendered as a standard HTML `<input type="date">` element. The display format of this element is controlled by the following system parameters:

- **Browser language**
- **Operating system regional settings**

To display and enter dates in a specific local format, respondents should ensure that both settings match their preferred region.

If you need to enforce a particular date format regardless of the user's local settings, consider one of the following approaches:

- **Input Mask**\
Apply a required format directly to a Single-Line Input field. See the [Masked Input Fields](https://surveyjs.io/form-library/examples/masked-input-fields/) demo for an example.

- **Third-Party Date Picker**\
Integrating a third-party date picker component gives you full control over both the display format and user interaction. Refer to the tutorials below for your framework of choice to learn how to integrate such components with SurveyJS:

  - [React](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-react)
  - [Angular](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-angular)
  - [Vue](https://surveyjs.io/form-library/documentation/customize-question-types/third-party-component-integration-vue)
