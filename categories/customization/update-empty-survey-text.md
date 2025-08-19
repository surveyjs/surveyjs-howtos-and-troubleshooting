## Problem
How to update the message `"The survey doesn't contain any visible elements."` when no JSON is found in the survey form library?

## Solution
You can customize the empty survey message by using the [`SurveyModel.emptySurveyText`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#emptySurveyText) property.

### Code Sample
```javascript
import { Model } from "survey-core";

const survey = new Model(json);
survey.emptySurveyText = "The form is empty";
```