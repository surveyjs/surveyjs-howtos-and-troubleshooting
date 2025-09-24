# Questionnaire Scoring for Medical Forms

## Problem
Doctors need to create tailored questionnaires for their practice without coding expertise, including the ability to calculate numerical scores based on patient responses to guide treatment plans. The scoring varies by form, such as the PHQ-9 for depression screening or custom true/false eligibility checks.

## Solution
You can use SurveyJS Creator to enable doctors to design questionnaires with built-in scoring logic. For numerical scoring (e.g., PHQ-9), assign values to radiogroup choices or matrix dropdowns and sum them using expressions. For conditional logic (e.g., eligibility screening), define calculated variables with initial values and update them based on triggers using expressions like `iif`. This approach requires no JavaScript coding and is configurable within the form designer.

#### Survey JSON Schema for PHQ-9 Scoring
Radiogroup questions with numerical values (0–3) summed via an expression for a total score.
```json
{
  "title": "PATIENT HEALTH QUESTIONNAIRE-9 (PHQ-9)",
  "pages": [
    {
      "name": "Page 1",
      "title": "Over the last 2 weeks, how often have you been bothered by any of the following problems?",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q1",
          "title": "Little interest or pleasure in doing things",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q2",
          "title": "Feeling down, depressed, or hopeless",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q3",
          "title": "Trouble falling or staying asleep, or sleeping too much",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q4",
          "title": "Feeling tired or having little energy",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q5",
          "title": "Poor appetite or overeating",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q6",
          "title": "Feeling bad about yourself — or that you are a failure or have let yourself or your family down",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q7",
          "title": "Trouble concentrating on things, such as reading the newspaper or watching television",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q8",
          "title": "Moving or speaking so slowly that other people could have noticed? Or the opposite — being so fidgety or restless that you have been moving around a lot more than usual",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q9",
          "title": "Thoughts that you would be better off dead or of hurting yourself in some way",
          "choices": [
            { "value": "0", "text": "Not at all" },
            { "value": "1", "text": "Several days" },
            { "value": "2", "text": "More than half the days" },
            { "value": "3", "text": "Nearly every day" }
          ]
        },
        {
          "type": "expression",
          "name": "totalScoreInfo",
          "title": "Total Score",
          "titleLocation": "left",
          "expression": "{q1} + {q2} + {q3} + {q4} + {q5} + {q6} + {q7} + {q8} + {q9}"
        }
      ]
    }
  ],
  "showQuestionNumbers": "on"
}
```

#### Survey JSON Schema for Eligibility Screening
A calculated variable `isIllegible` initialized to `true`. This variable is updated to `false` based on conditions (`q1='none' and q3=10 or q5='All'`) using the `iif` function.

```json
{
  "title": "Eligibility Screening Survey",
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q1",
          "title": "Q1. Select an option",
          "isRequired": true,
          "choices": [
            "none",
            "option1",
            "option2"
          ]
        },
        {
          "type": "text",
          "name": "q3",
          "title": "Q3. Enter a number",
          "isRequired": true,
          "inputType": "number"
        },
        {
          "type": "radiogroup",
          "name": "q5",
          "title": "Q5. Select one",
          "isRequired": true,
          "choices": [
            "All",
            "Some",
            "None"
          ]
        },
        {
          "type": "expression",
          "name": "isIllegibleResult",
          "title": "Patient Illegibility Status",
          "expression": "{eligibilityCheck}"
        }
      ]
    }
  ],
  "calculatedValues": [
    {
      "name": "isIllegible",
      "expression": "true"
    },
    {
      "name": "eligibilityCheck",
      "expression": "iif(({q1} = 'none' and {q3} = 10) or {q5} = 'All', false, {isIllegible})"
    }
  ]
}
```

## Learn More

* [SurveyJS Conditional Logic and Expression Functions](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#expressions)