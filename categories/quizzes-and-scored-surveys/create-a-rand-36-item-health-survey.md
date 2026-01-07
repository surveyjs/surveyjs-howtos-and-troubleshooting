# Create a RAND 36-Item Health Survey (SF-36) with Scoring in SurveyJS

The RAND 36-Item Short Form Health Survey (SF-36) is a widely validated instrument for assessing health-related quality of life. It consists of 36 standardized questions that measure eight core health domains, along with one health transition item.

This guide explains how to implement the RAND SF-36 questionnaire in SurveyJS and automatically calculate domain scores based on RAND's official scoring methodology.

## RAND SF-36 Scoring Overview

RAND SF-36 scoring converts raw questionnaire responses into standardized domain scores ranging from 0 to 100, where higher scores indicate better health status. The scoring process consists of two steps:

1. **Re-code individual items**\
Each response option is mapped to a predefined value between 0 and 100. Depending on the question, some items require reverse scoring.

2. **Average scores within each domain**\
Re-coded values for all items within a domain are averaged. Only answered items are included in the calculation, following the standard SF-36 prorating rule.

The SF-36 domains are:

* Physical Functioning (10 questions)
* Role Limitations &ndash; Physical Health (4 questions)
* Role Limitations &ndash; Emotional Problems (3 questions)
* Energy/Fatigue (Vitality) (4 questions)
* Emotional Well-Being (5 questions)
* Social Functioning (2 questions)
* Pain (2 questions)
* General Health (5 questions)
* Reported Health Transition (1 question)

Each domain produces a score from 0 to 100. For detailed scoring rules, see the official documentation: [RAND SF-36 Health Survey Scoring Instructions](https://www.rand.org/health/surveys/mos/36-item-short-form/scoring.html).

## Problem

Although the RAND SF-36 questionnaire uses standardized answer formats (for example, 3-point or 5-point Likert scales), the numeric score assigned to each answer varies by question. When implementing the questionnaire in SurveyJS Creator, this creates two challenges:

- Questions must be grouped into domains for scoring.
- Each choice option must carry a question-specific numeric score.

SurveyJS does not provide this functionality out of the box, so additional configuration is required.

## Solution

Use SurveyJS [Radio Button Group](https://surveyjs.io/form-library/examples/single-select-radio-button-group/) questions to represent SF-36 items. Extend the question's functionality with [custom properties](https://surveyjs.io/form-library/documentation/customize-question-types/add-custom-properties-to-a-form):

* `score` &mdash; A numeric value (0&ndash;100) assigned to an answer choice after re-coding.
* `domain` &mdash; A string identifier used to group questions into scoring domains.

On survey completion (using the [`onCompleting`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onCompleting) event), aggregate the scores for each domain, compute their averages, and store the results for display.

### Code Sample

```ts
import { Serializer } from "survey-core";

// Add `score` to choice options
Serializer.addProperty("itemvalue", {
  name: "score:number",
});

// Add `domain` to questions
Serializer.addProperty("question", "domain");

function calculateDomainScores(survey) {
  const scores = {}; // { domain: [score1, score2, ...] }

  survey.getAllQuestions().forEach((q) => {
    const value = survey.getValue(q.name);
    if (value !== undefined && q.domain && q.choices) {
      const selectedChoice = q.choices.find(c => c.value === value);
      if (selectedChoice && selectedChoice.score !== undefined) {
        if (!scores[q.domain]) scores[q.domain] = [];
        scores[q.domain].push(selectedChoice.score);
      }
    }
  });

  // Average only answered items (standard SF-36 scoring rule)
  const averages = {};
  Object.keys(scores).forEach(key => {
    const itemScores = scores[key];
    if (itemScores.length > 0) {
      averages[key] = Math.round(
        itemScores.reduce((a, b) => a + b, 0) / itemScores.length
      );
    } else {
      averages[key] = "N/A";
    }
  });

  return averages;
}

survey.onCompleting.add(() => {
  const domainScores = calculateDomainScores(survey);

  // Store scores in survey data for use in `completedHtml`
  Object.keys(domainScores).forEach(key => {
    survey.setValue(key, domainScores[key]);
  });
});
```

### JSON Schema for the RAND 36-Item Health Survey (SF-36)

Each question defines its own answer choices and scoring logic. The `domain` property assigns each question to the appropriate SF-36 domain, while the `score` property defines the re-coded numeric value for each answer option.

```json
{
  "title": "RAND 36-Item Health Survey 1.0",
  "description": "Choose one option for each questionnaire item.",
  "completedHtml": "<h3>Thank you for completing the survey!</h3><h4>Your SF-36 Domain Scores (0-100, higher = better health):</h4><ul><li><strong>Physical Functioning:</strong> {physical_functioning}</li><li><strong>Role Limitations - Physical Health:</strong> {role_physical}</li><li><strong>Role Limitations - Emotional Problems:</strong> {role_emotional}</li><li><strong>Energy/Fatigue (Vitality):</strong> {energy_fatigue}</li><li><strong>Emotional Well-Being:</strong> {emotional_wellbeing}</li><li><strong>Social Functioning:</strong> {social_functioning}</li><li><strong>Pain:</strong> {pain}</li><li><strong>General Health:</strong> {general_health}</li><li><strong>Reported Health Transition:</strong> {health_change}</li></ul>",
  "showTOC": true,
  "pages": [
    {
      "name": "general_intro",
      "title": "General Health",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q1",
          "title": "In general, would you say your health is:",
          "isRequired": true,
          "domain": "general_health",
          "choices": [
            { "value": 1, "text": "Excellent", "score": 100 },
            { "value": 2, "text": "Very good", "score": 75 },
            { "value": 3, "text": "Good", "score": 50 },
            { "value": 4, "text": "Fair", "score": 25 },
            { "value": 5, "text": "Poor", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q2",
          "title": "Compared to one year ago, how would you rate your health in general now?",
          "isRequired": true,
          "domain": "health_change",
          "choices": [
            { "value": 1, "text": "Much better now than one year ago", "score": 100 },
            { "value": 2, "text": "Somewhat better now than one year ago", "score": 75 },
            { "value": 3, "text": "About the same", "score": 50 },
            { "value": 4, "text": "Somewhat worse now than one year ago", "score": 25 },
            { "value": 5, "text": "Much worse now than one year ago", "score": 0 }
          ]
        }
      ]
    },
    {
      "name": "physical_functioning",
      "title": "Physical Functioning",
      "description": "The following items are about activities you might do during a typical day. Does your health now limit you in these activities? If so, how much?",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q3",
          "title": "Vigorous activities, such as running, lifting heavy objects, participating in strenuous sports",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q4",
          "title": "Moderate activities, such as moving a table, pushing a vacuum cleaner, bowling, or playing golf",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q5",
          "title": "Lifting or carrying groceries",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q6",
          "title": "Climbing several flights of stairs",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q7",
          "title": "Climbing one flight of stairs",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q8",
          "title": "Bending, kneeling, or stooping",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q9",
          "title": "Walking more than a mile",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q10",
          "title": "Walking several blocks",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q11",
          "title": "Walking one block",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q12",
          "title": "Bathing or dressing yourself",
          "isRequired": true,
          "domain": "physical_functioning",
          "choices": [
            { "value": 1, "text": "Yes, limited a lot", "score": 0 },
            { "value": 2, "text": "Yes, limited a little", "score": 50 },
            { "value": 3, "text": "No, not limited at all", "score": 100 }
          ]
        }
      ]
    },
    {
      "name": "role_physical",
      "title": "Role Limitations Due to Physical Health",
      "description": "During the past 4 weeks, have you had any of the following problems with your work or other regular daily activities as a result of your physical health?",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q13",
          "title": "Cut down the amount of time you spent on work or other activities",
          "isRequired": true,
          "domain": "role_physical",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q14",
          "title": "Accomplished less than you would like",
          "isRequired": true,
          "domain": "role_physical",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q15",
          "title": "Were limited in the kind of work or other activities",
          "isRequired": true,
          "domain": "role_physical",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q16",
          "title": "Had difficulty performing the work or other activities (for example, it took extra effort)",
          "isRequired": true,
          "domain": "role_physical",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        }
      ]
    },
    {
      "name": "role_emotional_social",
      "title": "Role Limitations Due to Emotional Problems & Social Functioning",
      "description": "During the past 4 weeks, have you had any of the following problems with your work or other regular daily activities as a result of any emotional problems (such as feeling depressed or anxious)?",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q17",
          "title": "Cut down the amount of time you spent on work or other activities",
          "isRequired": true,
          "domain": "role_emotional",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q18",
          "title": "Accomplished less than you would like",
          "isRequired": true,
          "domain": "role_emotional",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q19",
          "title": "Didn't do work or other activities as carefully as usual",
          "isRequired": true,
          "domain": "role_emotional",
          "choices": [
            { "value": 1, "text": "Yes", "score": 0 },
            { "value": 2, "text": "No", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q20",
          "title": "During the past 4 weeks, to what extent has your physical health or emotional problems interfered with your normal social activities with family, friends, neighbors, or groups?",
          "isRequired": true,
          "domain": "social_functioning",
          "choices": [
            { "value": 1, "text": "Not at all", "score": 100 },
            { "value": 2, "text": "Slightly", "score": 75 },
            { "value": 3, "text": "Moderately", "score": 50 },
            { "value": 4, "text": "Quite a bit", "score": 25 },
            { "value": 5, "text": "Extremely", "score": 0 }
          ]
        }
      ]
    },
    {
      "name": "pain",
      "title": "Pain",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q21",
          "title": "How much bodily pain have you had during the past 4 weeks?",
          "isRequired": true,
          "domain": "pain",
          "choices": [
            { "value": 1, "text": "None", "score": 100 },
            { "value": 2, "text": "Very mild", "score": 80 },
            { "value": 3, "text": "Mild", "score": 60 },
            { "value": 4, "text": "Moderate", "score": 40 },
            { "value": 5, "text": "Severe", "score": 20 },
            { "value": 6, "text": "Very severe", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q22",
          "title": "During the past 4 weeks, how much did pain interfere with your normal work (including both work outside the home and housework)?",
          "isRequired": true,
          "domain": "pain",
          "choices": [
            { "value": 1, "text": "Not at all", "score": 100 },
            { "value": 2, "text": "A little bit", "score": 75 },
            { "value": 3, "text": "Moderately", "score": 50 },
            { "value": 4, "text": "Quite a bit", "score": 25 },
            { "value": 5, "text": "Extremely", "score": 0 }
          ]
        }
      ]
    },
    {
      "name": "mental_health",
      "title": "Emotional Well-Being and Energy/Fatigue",
      "description": "These questions are about how you feel and how things have been with you during the past 4 weeks. For each question, please give the one answer that comes closest to the way you have been feeling.\n\nHow much of the time during the past 4 weeks...",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q23",
          "title": "Did you feel full of pep?",
          "isRequired": true,
          "domain": "energy_fatigue",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 100 },
            { "value": 2, "text": "Most of the time", "score": 80 },
            { "value": 3, "text": "A good bit of the time", "score": 60 },
            { "value": 4, "text": "Some of the time", "score": 40 },
            { "value": 5, "text": "A little of the time", "score": 20 },
            { "value": 6, "text": "None of the time", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q24",
          "title": "Have you been a very nervous person?",
          "isRequired": true,
          "domain": "emotional_wellbeing",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 0 },
            { "value": 2, "text": "Most of the time", "score": 20 },
            { "value": 3, "text": "A good bit of the time", "score": 40 },
            { "value": 4, "text": "Some of the time", "score": 60 },
            { "value": 5, "text": "A little of the time", "score": 80 },
            { "value": 6, "text": "None of the time", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q25",
          "title": "Have you felt so down in the dumps that nothing could cheer you up?",
          "isRequired": true,
          "domain": "emotional_wellbeing",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 0 },
            { "value": 2, "text": "Most of the time", "score": 20 },
            { "value": 3, "text": "A good bit of the time", "score": 40 },
            { "value": 4, "text": "Some of the time", "score": 60 },
            { "value": 5, "text": "A little of the time", "score": 80 },
            { "value": 6, "text": "None of the time", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q26",
          "title": "Have you felt calm and peaceful?",
          "isRequired": true,
          "domain": "emotional_wellbeing",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 100 },
            { "value": 2, "text": "Most of the time", "score": 80 },
            { "value": 3, "text": "A good bit of the time", "score": 60 },
            { "value": 4, "text": "Some of the time", "score": 40 },
            { "value": 5, "text": "A little of the time", "score": 20 },
            { "value": 6, "text": "None of the time", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q27",
          "title": "Did you have a lot of energy?",
          "isRequired": true,
          "domain": "energy_fatigue",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 100 },
            { "value": 2, "text": "Most of the time", "score": 80 },
            { "value": 3, "text": "A good bit of the time", "score": 60 },
            { "value": 4, "text": "Some of the time", "score": 40 },
            { "value": 5, "text": "A little of the time", "score": 20 },
            { "value": 6, "text": "None of the time", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q28",
          "title": "Have you felt downhearted and blue?",
          "isRequired": true,
          "domain": "emotional_wellbeing",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 0 },
            { "value": 2, "text": "Most of the time", "score": 20 },
            { "value": 3, "text": "A good bit of the time", "score": 40 },
            { "value": 4, "text": "Some of the time", "score": 60 },
            { "value": 5, "text": "A little of the time", "score": 80 },
            { "value": 6, "text": "None of the time", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q29",
          "title": "Did you feel worn out?",
          "isRequired": true,
          "domain": "energy_fatigue",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 0 },
            { "value": 2, "text": "Most of the time", "score": 20 },
            { "value": 3, "text": "A good bit of the time", "score": 40 },
            { "value": 4, "text": "Some of the time", "score": 60 },
            { "value": 5, "text": "A little of the time", "score": 80 },
            { "value": 6, "text": "None of the time", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q30",
          "title": "Have you been a happy person?",
          "isRequired": true,
          "domain": "emotional_wellbeing",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 100 },
            { "value": 2, "text": "Most of the time", "score": 80 },
            { "value": 3, "text": "A good bit of the time", "score": 60 },
            { "value": 4, "text": "Some of the time", "score": 40 },
            { "value": 5, "text": "A little of the time", "score": 20 },
            { "value": 6, "text": "None of the time", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q31",
          "title": "Did you feel tired?",
          "isRequired": true,
          "domain": "energy_fatigue",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 0 },
            { "value": 2, "text": "Most of the time", "score": 20 },
            { "value": 3, "text": "A good bit of the time", "score": 40 },
            { "value": 4, "text": "Some of the time", "score": 60 },
            { "value": 5, "text": "A little of the time", "score": 80 },
            { "value": 6, "text": "None of the time", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q32",
          "title": "During the past 4 weeks, how much of the time has your physical health or emotional problems interfered with your social activities (like visiting with friends, relatives, etc.)?",
          "isRequired": true,
          "domain": "social_functioning",
          "choices": [
            { "value": 1, "text": "All of the time", "score": 0 },
            { "value": 2, "text": "Most of the time", "score": 25 },
            { "value": 3, "text": "Some of the time", "score": 50 },
            { "value": 4, "text": "A little of the time", "score": 75 },
            { "value": 5, "text": "None of the time", "score": 100 }
          ]
        }
      ]
    },
    {
      "name": "general_health_perceptions",
      "title": "General Health Perceptions",
      "description": "How TRUE or FALSE is each of the following statements for you?",
      "elements": [
        {
          "type": "radiogroup",
          "name": "q33",
          "title": "I seem to get sick a little easier than other people",
          "isRequired": true,
          "domain": "general_health",
          "choices": [
            { "value": 1, "text": "Definitely true", "score": 0 },
            { "value": 2, "text": "Mostly true", "score": 25 },
            { "value": 3, "text": "Don't know", "score": 50 },
            { "value": 4, "text": "Mostly false", "score": 75 },
            { "value": 5, "text": "Definitely false", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q34",
          "title": "I am as healthy as anybody I know",
          "isRequired": true,
          "domain": "general_health",
          "choices": [
            { "value": 1, "text": "Definitely true", "score": 100 },
            { "value": 2, "text": "Mostly true", "score": 75 },
            { "value": 3, "text": "Don't know", "score": 50 },
            { "value": 4, "text": "Mostly false", "score": 25 },
            { "value": 5, "text": "Definitely false", "score": 0 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q35",
          "title": "I expect my health to get worse",
          "isRequired": true,
          "domain": "general_health",
          "choices": [
            { "value": 1, "text": "Definitely true", "score": 0 },
            { "value": 2, "text": "Mostly true", "score": 25 },
            { "value": 3, "text": "Don't know", "score": 50 },
            { "value": 4, "text": "Mostly false", "score": 75 },
            { "value": 5, "text": "Definitely false", "score": 100 }
          ]
        },
        {
          "type": "radiogroup",
          "name": "q36",
          "title": "My health is excellent",
          "isRequired": true,
          "domain": "general_health",
          "choices": [
            { "value": 1, "text": "Definitely true", "score": 100 },
            { "value": 2, "text": "Mostly true", "score": 75 },
            { "value": 3, "text": "Don't know", "score": 50 },
            { "value": 4, "text": "Mostly false", "score": 25 },
            { "value": 5, "text": "Definitely false", "score": 0 }
          ]
        }
      ]
    }
  ],
  "showQuestionNumbers": true,
  "completeText": "Submit Survey",
  "requiredMark": "(required)"
}
```

### Live Demo 

[View in Plunker](https://plnkr.co/edit/8Ra1GMeRIKJ9ehln)