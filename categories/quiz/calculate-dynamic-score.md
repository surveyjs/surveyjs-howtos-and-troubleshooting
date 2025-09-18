# Dynamic Score Calculation in Surveys

## Problem
How to implement a custom variable in a survey that dynamically calculates a total score based on user responses and include this score in the dataset upon survey completion? The score should be calculated dynamically, and both individual answers and the total score should be included in the response data.

## Solution
To achieve a dynamically calculated score in a survey using SurveyJS, register a custom `"score"` property for `ItemValue` (used for choices and single-select matrix columns) using the `Survey.Serializer.addProperty` function to ensure the property is properly recognized and serialized. Then, in the [`SurveyModel.onCompleting`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onCompleting) event, iterate through all questions, retrieve the selected value, find the corresponding options's `score`, and compute the total. Use [`SurveyModel.setValue`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#setValue) to store the `totalScore` in the survey [`data`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#data) to include it in the response alongside individual answers. Below is the corrected example with the required property registration.

### Code Sample
```javascript
import { Serializer } from "survey-core";

// Register the custom 'score' property for ItemValue (choices and single-select matrix column)
Serializer.addProperty("itemvalue", {
    name: "score",
    type: "number",
    default: 0
});

// Initialize the survey
var survey = new Survey.Model(surveyJson);

// Calculate total score on survey completion
survey.onCompleting.add(function(sender, options) {
  var totalScore = 0;
  
  // Iterate through all questions
  sender.getAllQuestions().forEach(function(question) {
    var value = sender.getValue(question.name);
    if (value !== undefined && value !== null) {
      // Find the selected choice and its score
      var choice = question.visibleChoices.find(function(item) {
        return item.value === value;
      });
      if (choice && typeof choice.score !== 'undefined') {
        totalScore += choice.score;
      }
    }
  });
  
  // Store the total score in the survey data
  sender.setValue("totalScore", totalScore);
});

// Handle survey completion to ensure data includes totalScore
survey.onComplete.add(function(sender, options) {
  console.log("Survey Results:", sender.data);
  // Example output: { q1: 2, q2: 3, totalScore: 50 }
});

// Render the survey
return (<Survey model={survey} />);
```

### Survey JSON Schema
Create a survey and define the `score` property within individual choices and matrix columns.
```json
{
  "pages": [
    {
      "elements": [
        {
          "type": "radiogroup",
          "name": "q1",
          "title": "How satisfied are you with our service?",
          "choices": [
            { "value": 1, "text": "Not satisfied", "score": 10 },
            { "value": 2, "text": "Somewhat satisfied", "score": 20 },
            { "value": 3, "text": "Very satisfied", "score": 30 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q2",
          "title": "How likely are you to recommend us?",
          "choices": [
            { "value": 1, "text": "Not likely", "score": 10 },
            { "value": 2, "text": "Neutral", "score": 20 },
            { "value": 3, "text": "Very likely", "score": 30 }
          ]
        }
      ]
    }
  ]
}
```

## Learn More
For a live example, view the [Scored Survey Demo](https://surveyjs.io/form-library/examples/create-a-scored-survey/). This demo shows how to calculate a `totalScore` based on custom `score` properties attached to each question's choices and matrix columns and include it in the survey response data. For additional details on registering custom properties, refer to the [SurveyJS Documentation](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form).