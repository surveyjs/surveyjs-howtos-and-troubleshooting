# How to Render Markdown Static Texts in a Survey

## Problem

You want to display rich formatted instructions, descriptions, or comments in a survey using Markdown. Survey fields should support dynamic values like user name, date, or ID, and render properly in a PDF or web interface.

## Solution

Use SurveyJS for survey definitions and a third-party Markdown-to-HTML JavaScript converter to convert Markdown text into HTML for rendering. This allows you to use bold, italic, and inline dynamic values while keeping the survey business-ready.

Steps:

1. Define your survey JSON and include Markdown-enabled descriptions.
2. Optionally: Use expression fields for dynamic values like current date.
3. Use a third-party Markdown-to-HTML JavaScript converter to convert Markdown to HTML when rendering. In this demo, markdownit is used.
4. Optionally sanitize the HTML before displaying it.

### Code Sample

```javascript
import markdownit from "markdown-it";

const converter = markdownit({
  html: true, // Support HTML tags in the source
});

const surveyPDF = new SurveyPDF(json, options);

// Include custom values in a survey PDF document
surveyPDF.setValue("id", "123");
surveyPDF.setValue("userName", "User Name");

surveyPDF.onTextMarkdown.add((_, options) => {
  // Convert Markdown to HTML
  let str = converter.renderInline(options.text);
  // Optionally sanitize the HTML markup here
  options.html = str;
});
```

### Survey JSON Schema

```
{
  title: "User Feedback Survey",
  description: `Name: **{userName}**. Today's date: **{todaysDate}**. ID: **{id}**`,
  pages: [
    {
      name: "user_info",
      elements: [
        {
          type: "expression",
          name: "todaysDate",
          visible: false,
          expression: "currentDate()",
          displayStyle: "date"
        },
        {
          type: "text",
          name: "name",
          title: "Name",
          description: "Enter your full name.",
          isRequired: true
        },
        {
          type: "text",
          name: "label",
          title: "Label",
          description: "Enter the relevant label for your role or department."
        },
        {
          type: "text",
          name: "date",
          title: "Date",
          description: "Select today's date.",
          inputType: "date"
        },
        {
          type: "text",
          name: "id",
          title: "ID",
          description: "Enter your unique ID."
        }
      ]
    },
    {
      name: "feedback",
      elements: [
        {
          type: "comment",
          name: "comments",
          title: "Please provide your feedback",
          description: "Share your thoughts about our product or service. Markdown is supported here as well."
        },
        {
          type: "rating",
          name: "satisfaction",
          title: "Overall satisfaction",
          minRateDescription: "Very dissatisfied",
          maxRateDescription: "Very satisfied"
        }
      ]
    }
  ]
}
```

## Live Demo

[View in CodeSandbox](https://codesandbox.io/p/sandbox/pensive-noether-gdk8sh?file=%2Fsrc%2Findex.js%3A28%2C3-39%2C6)

## Learn More

- [markdown-it](https://github.com/markdown-it/markdown-it)
- [Convert Markdown to HTML with markdown-it](https://surveyjs.io/form-library/examples/edit-survey-questions-markdown/)
- [Convert Markdown to HTML with Marked](https://surveyjs.io/form-library/examples/enable-markdown-in-surveys/)