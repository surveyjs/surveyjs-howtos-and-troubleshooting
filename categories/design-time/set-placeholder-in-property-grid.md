# Set a Placeholder for a Property in the Property Grid (SurveyJS Creator)

## Problem

I want to add a placeholder to a property in the Property Grid to guide users on what values they should enter.

## Solution

Placeholders can be defined using [localization strings](https://surveyjs.io/survey-creator/documentation/survey-localization-translate-surveys-to-different-languages#override-individual-translations). Access the locale strings for the target language (for example, English) and set the placeholder under the `peplaceholder` object (`pe` stands for "property editor").

In the example below, a placeholder is set for the `description` property of survey questions (via the `question` object). You can nest placeholders under other objects to scope them to specific element types.

### Code Example

```js
import { getLocaleStrings } from "survey-creator-core";

// Get the English locale strings
const enLocale = getLocaleStrings("en");

// Ensure the `question` placeholder object exists
if (!enLocale.peplaceholder.question) {
  enLocale.peplaceholder.question = {};
}

// Define the placeholder for the `description` property of questions
enLocale.peplaceholder.question.description = "Enter the question description...";
```