# How to Disable In-Place Editing for Specific SurveyJS Question Elements

## Problem
In SurveyJS Creator, you may want to prevent users from editing certain parts of a question, such as the question title, description, or choice text. By default, SurveyJS allows inline editing of these elements, but sometimes it’s necessary to make them read-only to maintain consistency or control the survey design.

## Solution
You can achieve this by using the [`creator.onAllowInplaceEdit`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#onAllowInplaceEdit) event. This event allows you to control which elements can be edited in-place. By checking the property being edited, you can selectively disable editing for titles, descriptions, or any other fields.

### Code Sample
```javascript
// Define the elements you want to make read-only
const readOnlyInplaceEditors = ["title", "description", "text"];

// Add event handler to disable inline editing for specific properties
creator.onAllowInplaceEdit.add((sender, options) => {
  if (readOnlyInplaceEditors.indexOf(options.propertyName) >= 0) {
    options.allow = false;
  }
});
```

### Survey JSON Schema

```json
{
  "title": "Sample Survey",
  "description": "<ul><li><strong>Bold</strong></li><li><em>Italics</em></li><li><s>Strikethrough</s></li></ul>",
  "pages": [
    {
      "name": "page1",
      "elements": [
        {
          "type": "quill",
          "name": "rich-text-editor",
          "title": "<p>Rich&nbsp;Text&nbsp;Editor</p>",
          "defaultValue": "Hello world!"
        },
        {
          "type": "text",
          "name": "question1",
          "title": "<h2><strong><em>Question&nbsp;Title</em></strong></h2>",
          "description": "<h3><strong><em><u>Question&nbsp;Description</u></em></strong></h3>"
        },
        {
          "type": "radiogroup",
          "name": "question2",
          "choices": [
            {
              "value": "item1",
              "text": "<strong>Item 1</strong>"
            },
            {
              "value": "item2",
              "text": "<i>Item 2</i>"
            },
            "Item 3"
          ]
        }
      ]
    }
  ]
}
```

### Live demo

[CodeSandbox Example](https://codesandbox.io/p/sandbox/sad-keldysh-txr5pn?file=%2Fpackage.json%3A9%2C30)