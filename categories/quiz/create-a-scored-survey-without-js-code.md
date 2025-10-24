# Create a Scored Questionnaire Without JavaScript Code in SurveyJS

## Problem

Doctors often need to build custom questionnaires for their practice without coding. These forms may need to calculate scores from patient responses to support treatment decisions. Scoring rules vary: for example, the PHQ-9 for depression screening uses summed values, while eligibility checks rely on true/false conditions.

## Solution

Even without JavaScript, you can implement scoring in two ways:

- **Multiple-choice scoring (dropdowns, radio groups)**      
Assign numeric values to answer options and use an expression to sum them into a total score. See [Survey JSON Schema for PHQ-9 Scoring](#survey-json-schema-for-phq-9-scoring).

- **Calculated values**      
For yes/no eligibility rules, create a calculated value with an expression that evaluates conditions (e.g., "age over 18 and not pregnant"). See [Survey JSON Schema for Eligibility Screening](#survey-json-schema-for-eligibility-screening).

### Survey JSON Schema for PHQ-9 Scoring

```json
{
  "title": "PATIENT HEALTH QUESTIONNAIRE-9 (PHQ-9)",
  "pages": [
    {
      "name": "page1",
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
          "choicesFromQuestion": "q1"
        },
        {
          "type": "radiogroup",
          "name": "q3",
          "title": "Trouble falling or staying asleep, or sleeping too much",
          "choicesFromQuestion": "q1"

        },
        {
          "type": "radiogroup",
          "name": "q4",
          "title": "Feeling tired or having little energy",
          "choicesFromQuestion": "q1"
        },
        {
          "type": "radiogroup",
          "name": "q5",
          "title": "Poor appetite or overeating",
          "choicesFromQuestion": "q1"
        },
        {
          "type": "radiogroup",
          "name": "q6",
          "title": "Feeling bad about yourself — or that you are a failure or have let yourself or your family down",
          "choicesFromQuestion": "q1"
        },
        {
          "type": "radiogroup",
          "name": "q7",
          "title": "Trouble concentrating on things, such as reading the newspaper or watching television",
          "choicesFromQuestion": "q1"
        },
        {
          "type": "radiogroup",
          "name": "q8",
          "title": "Moving or speaking so slowly that other people could have noticed? Or the opposite — being so fidgety or restless that you have been moving around a lot more than usual",
          "choicesFromQuestion": "q1"
        },
        {
          "type": "radiogroup",
          "name": "q9",
          "title": "Thoughts that you would be better off dead or of hurting yourself in some way",
          "choicesFromQuestion": "q1"
        },
        {
          "type": "expression",
          "name": "totalScoreInfo",
          "title": "Total Score: ",
          "titleLocation": "left",
          "showNumber": false,
          "expression": "{q1} + {q2} + {q3} + {q4} + {q5} + {q6} + {q7} + {q8} + {q9}"
        }
      ]
    }
  ],
  "showQuestionNumbers": true
}
```

[Open in Plunker](https://plnkr.co/edit/GiNUr7uq0PfqCftM)

### Survey JSON Schema for Eligibility Screening

<!-- TODO: Update the schema after https://github.com/surveyjs/survey-library/issues/10414 -->

```json
{
  "title": "Medical Eligibility Screening Survey",
  "description": "Please answer the following questions honestly to help us determine your eligibility.",
  "calculatedValues": [
    {
      "name": "isEligible",
      "expression": "age({dob}) >= 18 and !{pregnant} and !({conditions} anyof ['Heart disease','Cancer','Immunodeficiency'])",
      "includeIntoResult": true
    }
  ],
  "pages": [
    {
      "name": "basic_info",
      "elements": [
        {
          "type": "text",
          "name": "fullName",
          "title": "Full Name",
          "isRequired": true
        },
        {
          "type": "text",
          "inputType": "date",
          "name": "dob",
          "title": "Date of Birth",
          "isRequired": true
        },
        {
          "type": "radiogroup",
          "name": "gender",
          "title": "Gender",
          "isRequired": true,
          "choices": ["Male", "Female", "Other", "Prefer not to say"]
        }
      ]
    },
    {
      "name": "medical_history",
      "elements": [
        {
          "type": "checkbox",
          "name": "conditions",
          "title": "Do you have or have you ever been diagnosed with any of the following?",
          "isRequired": true,
          "choices": [
            "Heart disease",
            "High blood pressure",
            "Diabetes",
            "Asthma or other respiratory condition",
            "Cancer",
            "Immunodeficiency"
          ],
          "showNoneItem": true,
          "noneText": "None of the above",
        },
        {
          "type": "boolean",
          "name": "pregnant",
          "title": "Are you currently pregnant?",
          "visibleIf": "{gender} = 'Female'"
        },
        {
          "type": "boolean",
          "name": "medications",
          "title": "Are you currently taking any prescription medications?"
        },
        {
          "type": "comment",
          "name": "medications_list",
          "title": "If yes, please list them",
          "visibleIf": "{medications} = true"
        }
      ]
    },
    {
      "name": "risk_factors",
      "elements": [
        {
          "type": "boolean",
          "name": "allergies",
          "title": "Do you have any severe allergies (e.g., to medication, food, latex)?",
          "isRequired": true
        },
        {
          "type": "boolean",
          "name": "smoking",
          "title": "Do you currently smoke or use tobacco products?"
        },
        {
          "type": "boolean",
          "name": "alcohol",
          "title": "Do you consume alcohol regularly?"
        },
        {
          "type": "boolean",
          "name": "drug_use",
          "title": "Do you use recreational drugs?"
        }
      ]
    },
    {
      "name": "final",
      "elements": [
        {
          "type": "comment",
          "name": "other_concerns",
          "title": "Please list any other health concerns you think we should be aware of"
        },
        {
          "type": "expression",
          "name": "eligibility_result",
          "title": "Eligibility Result",
          "showNumber": false,
          "expression": "iif({isEligible} = true, '✅ You are eligible.', '❌ You are not eligible.')"
        }
      ]
    }
  ],
  "showQuestionNumbers": true,
  "completeText": "Submit"
}
```

<!-- TODO: Update Plunker after https://github.com/surveyjs/survey-library/issues/10414 -->

[Open in Plunker](https://plnkr.co/edit/Bu1pVNmnU63rU7Od)

## Learn More

* [SurveyJS Expressions](https://surveyjs.io/form-library/documentation/design-survey/conditional-logic#expressions)