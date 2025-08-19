# Customize the Empty Survey Placeholder in SurveyJS

## Problem

By default, SurveyJS Form Library displays a placeholder message when a survey has no visible elements. I want to change this message to better match my application's tone or context.

## Solution

You can update the placeholder text by setting the [`emptySurveyText`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#emptySurveyText) property of the `SurveyModel`.

### Code Sample

```javascript
import { Model } from "survey-core";

const survey = new Model(json);
survey.emptySurveyText = "The form is empty.";
```