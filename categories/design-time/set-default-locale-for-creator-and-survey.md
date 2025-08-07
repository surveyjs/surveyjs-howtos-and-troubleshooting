# Set Default Locale for SurveyJS Creator and Survey

## Problem
How to set the default locale for both SurveyJS Creator (Editor) and Survey to German?

## Solution
To set the UI locale of SurveyJS Creator to German, configure the [`creator.locale`](https://surveyjs.io/survey-creator/documentation/survey-localization-translate-surveys-to-different-languages) property. To make German the default locale for all surveys created in the editor, update the [`surveyLocalization.defaultLocale`](https://surveyjs.io/form-library/documentation/survey-localization) setting in the SurveyJS Form Library. This ensures German text is saved under the "default" key, while still allowing survey editors to translate into other languages via the Translation tab or by adjusting the survey's locale.

## Code Sample
```javascript
// Set SurveyJS Creator UI locale to German
creator.locale = "de";

// Set default locale for all surveys to German
import { surveyLocalization } from "survey-core";
surveyLocalization.defaultLocale = "de";
```