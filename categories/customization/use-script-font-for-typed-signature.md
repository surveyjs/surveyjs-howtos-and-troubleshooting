# Use a Script Font for Typed Signatures in SurveyJS

## Problem

When the same person signs multiple documents, signatures drawn with a mouse or trackpad often vary from one document to another. This inconsistency can be undesirable in legal, compliance, or business workflows that require a uniform signature appearance.

I need a way to let respondents type their name and display it in a consistent script-style font instead of capturing a handwritten signature.

## Solution

Use a [Single-Line Input](https://surveyjs.io/form-library/documentation/api-reference/text-entry-question-model) question instead of a Signature Pad. Handle the [`onUpdateQuestionCssClasses`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onUpdateQuestionCssClasses) event to assign a custom CSS class to the question, then style the input with a script font such as [Great Vibes](https://fonts.google.com/specimen/Great+Vibes).

Respondents type their name, and the input renders it in a cursive font that resembles a handwritten signature while maintaining a consistent appearance across all documents.

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

Assign a custom CSS class to the signature question so the styles affect only its input field:

```javascript
// ...
// Omitted: `Survey.Model` creation
// ...
survey.onUpdateQuestionCssClasses.add((_, options) => {
  if (options.question.name === "signature") {
    options.cssClasses.root += " signature-input";
  }
});
```

### Style the Input with a Script Font

Import a script font and apply it to the custom CSS class:

```css
@import url('https://fonts.googleapis.com/css2?family=Great+Vibes&display=swap');

.signature-input input {
  font-family: 'Great Vibes', cursive;
  font-size: 24px;
}
```

[Open in Plunker](https://plnkr.co/edit/pv3IwxvCbZ9HFsHn)

### Let Respondents Choose a Signature Method (Optional)

If you want to support both handwritten and typed signatures, add a choice question that lets respondents select their preferred signing method and display the appropriate input:

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

Apply the custom CSS class only to the typed signature question by using the `onUpdateQuestionCssClasses` event handler shown above.

[Open in Plunker](https://plnkr.co/edit/z9r60dy35vlz6KTN)