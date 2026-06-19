# Use a Script Font for Typed Signatures in SurveyJS

## Problem

When multiple documents are signed in sequence by the same person, signatures drawn with a mouse or trackpad often look different on each document. This inconsistency can be problematic in legal or compliance workflows where a uniform signature appearance is expected.

You need a way to let respondents type their name and display it in a consistent script-style font, instead of capturing a freehand image signature.

## Solution

Use a [Single-Line Input](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model) question for the signature field. Handle the [`onUpdateQuestionCssClasses`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUpdateQuestionCssClasses) event to attach a custom CSS class to that question, then style the input with a widely available web font (for example, [Great Vibes](https://fonts.google.com/specimen/Great+Vibes) from Google Fonts).

Respondents type their name; the input renders it in a cursive font that looks like a script signature and stays consistent across every document.

### Survey JSON Schema

Add a Single-Line Input question for the typed signature:

```json
{
  "type": "text",
  "name": "signature",
  "title": "Sign by typing your full name",
  "description": "Your name will appear in a script font as your signature."
}
```

### Apply a Custom CSS Class

Assign a custom class to the signature question so styles apply only to that field:

```javascript
// ...
// Omitted: `Survey.Model` creation
// ...
survey.onUpdateQuestionCssClasses.add((sender, options) => {
  if (options.question.name === "signature") {
    options.cssClasses.root += " signatureInput";
  }
});
```

### Style the Input with a Script Font

Load an accessible web font and scope styles to the custom class. Great Vibes is available through Google Fonts and renders on most modern browsers and devices:

```css
@import url('https://fonts.googleapis.com/css2?family=Great+Vibes&display=swap');

.signatureInput input {
  font-family: 'Great Vibes', cursive;
  font-size: 24px;
}
```

### Optional: Let Respondents Choose a Signature Method

To offer both a signature pad and a typed script signature, add a choice question and show the appropriate field based on the selection:

```json
{
  "elements": [
    {
      "type": "radiogroup",
      "name": "signatureMethod",
      "title": "How would you like to sign?",
      "choices": [
        { "value": "pad", "text": "Draw my signature" },
        { "value": "typed", "text": "Type my name" }
      ]
    },
    {
      "type": "signaturepad",
      "name": "signaturePad",
      "title": "Draw your signature",
      "visibleIf": "{signatureMethod} = 'pad'"
    },
    {
      "type": "text",
      "name": "signature",
      "title": "Type your full name",
      "visibleIf": "{signatureMethod} = 'typed'"
    }
  ]
}
```

Apply the script font only to the typed signature question using the `onUpdateQuestionCssClasses` handler shown above.

### Live Demos

- [Text entry with script font styling](https://plnkr.co/edit/pv3IwxvCbZ9HFsHn)
- [Choice between signature pad and typed name](https://plnkr.co/edit/z9r60dy35vlz6KTN)
