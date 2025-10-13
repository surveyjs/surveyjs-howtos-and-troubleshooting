# Excluding Custom Question from Progress Bar

## Problem
When using a form progress bar with the type set to “answered questions” or “answered required questions,” the progress bar includes the captcha question in the progress calculation.

## Solution
If a custom question doesn't have an answer, you can create a custom question model that inherits from the `QuestionNonValue` class. Questions derived from `QuestionNonValue` are automatically excluded from the progress indication. A demo of a question extending `QuestionNonValue` is available at [Implement a Descriptive Text Element](https://surveyjs.io/survey-creator/examples/custom-descriptive-text-element/).

If you are extending a regular `Question` class, override the `hasInput` function to return `false`. This will exclude the custom question from the progress information. A question’s progress information is determined by the `hasInput` property, which is evaluated within the `getProgressInfo` function.

### Code Sample
```javascript
get hasInput(): boolean {
    return false;
}
```

Alternatively, to exclude the question from the total number of questions in the progress bar, implement the [`SurveyModel.onProgressText`](https://github.com/surveyjs/survey-library/blob/86dc0416b36422955f9998e4464c3416943e730a/packages/survey-core/src/question.ts#L571-L579) function and modify `options.text` to reduce the total question count.