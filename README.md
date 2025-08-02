# SurveyJS Q&A

Welcome to **SurveyJS Q&A**: Your go-to resource for community-driven questions and answers about [SurveyJS](https://surveyjs.io/), a powerful JavaScript form builder library. Whether you're a developer integrating SurveyJS with React, Angular, or Vue, or a survey creator designing dynamic JSON-driven forms, this repository offers **code snippets**, **survey JSON examples**, and practical solutions to help you succeed.

## Purpose
This repository is a centralized hub for:
- **Developers**: Find code snippets for integrating SurveyJS with frameworks, customizing question types, or troubleshooting issues.
- **Survey Creators**: Access survey JSON examples to build dynamic forms without coding, using SurveyJS Creator.
- **Community**: Share knowledge, ask questions, and contribute answers to help others in the SurveyJS ecosystem.

## Repository Structure
The content is organized for easy navigation and searchability:

- **/categories/integration/**: Q&A on integrating SurveyJS with frameworks like React, Angular, Vue, etc.
- **/categories/customization/**: Solutions for custom widgets, themes, and conditional logic.
- **/categories/json-examples/**: Survey JSON files and explanations for creating forms in SurveyJS Creator.
- **/categories/troubleshooting/**: Fixes for common errors and debugging tips.
- **/categories/advanced/**: Advanced topics like API extensions and performance optimization.
- **index.md**: A table of all Q&A entries with categories, titles, and tags for quick reference.
- **assets/**: Supporting images or reusable code snippets (if needed).

Browse the [index.md](./index.md) for a complete list of questions and answers.

## How to Use
- **Search**: Use GitHub’s search bar with keywords like "React integration" or "survey JSON" to find relevant Q&A.
- **Browse by Category**: Explore the `/categories/` folders for specific topics.
- **Copy JSON**: Use JSON files in `/categories/json-examples/` directly in SurveyJS Creator.
- **Contribute**: Add your own Q&A by following the [CONTRIBUTING.md](./CONTRIBUTING.md) guidelines.

## Example Q&A
### Developer Example
**Question**: How do I integrate SurveyJS with React?  
**Answer**: Install `survey-react-ui`, import the `Survey` component, and render with a JSON model.  
```jsx
import { Model } from 'survey-core';
import { Survey } from 'survey-react-ui';
const surveyJson = { /* Your survey JSON */ };
const survey = new Model(surveyJson);
return <Survey model={survey} />;
```
[See full Q&A](./categories/integration/how-to-integrate-surveyjs-with-react.md)

### Survey Creator Example
**Question**: How do I create a basic feedback survey?  
**Answer**: Use this JSON in SurveyJS Creator:  
```json
{
  "pages": [
    {
      "elements": [
        { "type": "text", "name": "name", "title": "Your Name" }
      ]
    }
  ]
}
```
[See full example](./categories/json-examples/basic-survey-form.md)

## Contributing
We welcome contributions from the SurveyJS community! To add a new Q&A:
1. Follow the template in [CONTRIBUTING.md](./CONTRIBUTING.md).
2. Submit a pull request with your Q&A in the appropriate category.
3. Ensure code snippets are tested and JSON examples are valid.

## Related Resources
- [SurveyJS Official Website](https://surveyjs.io/)
- [SurveyJS Documentation](https://surveyjs.io/documentation)
- [SurveyJS Creator Demo](https://surveyjs.io/create-survey)
- [GitHub Issues](https://github.com/surveyjs/survey-qa/issues): Request new Q&A or report issues.

## License
This repository is licensed under the [MIT License](./LICENSE).

---

⭐ **Star this repo** to support the SurveyJS community!  
📢 Share your questions or solutions via pull requests or issues.
