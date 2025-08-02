# Register Custom Location Question in SurveyJS Toolbox Under Text Category

## Question
How do I create and register a custom "location" question in SurveyJS that captures latitude and longitude, and place it in the toolbox under the "Text" category alongside standard text questions like "text," "comment," and "multipletext"?

## Answer
To create a custom "location" question in SurveyJS, use the `ComponentCollection` to register a new component based on the "multipletext" type. This allows you to define fields for latitude and longitude. Then, in the Survey Creator, change the category of this custom question to "text" to group it with built-in text questions (text, comment, multipletext).

### Step 1: Register the Custom Component
Import `ComponentCollection` from "survey-core" and add the custom question definition. The `questionJSON` uses "multipletext" as the base type with items for "lat" and "lng."

```typescript
import { ComponentCollection } from "survey-core";

ComponentCollection.Instance.add({
  name: "location",
  title: "Location",
  iconName: "icon-textedit-24x24", // Replace with your custom icon name if needed
  defaultQuestionTitle: "Location",
  questionJSON: {
    type: "multipletext",
    items: [
      { name: "lat", title: "Latitude" },
      { name: "lng", title: "Longitude" }
    ]
  }
});
```

### Step 2: Change Category in Survey Creator Toolbox
After registering the component, create a `SurveyCreator` instance and use `changeCategory` to move the "location" question to the "text" category.

```typescript
import { SurveyCreator } from "survey-creator-core"; // Assuming you're using Survey Creator

const creator = new SurveyCreator();
creator.toolbox.changeCategory("location", "text");
```

**Notes**:
- The "text" category in SurveyJS toolbox includes questions like "text," "comment," and "multipletext." This ensures your custom "location" question appears there for easy access.
- Customize the `iconName` if you have a specific icon; otherwise, it defaults to a text-related icon.
- This setup works in both runtime surveys and the Survey Creator UI.

**Pros**: Leverages existing "multipletext" for simplicity; easy categorization for better UX in Creator.  
**Cons**: If you need more advanced features (e.g., geolocation integration), extend the component further with custom logic.

## Recommended Approach
This method is ideal for developers extending SurveyJS with reusable custom questions. Test in your application to ensure the category change applies correctly.

## Related Tags
- surveyjs
- custom-question
- toolbox
- multipletext
- survey-creator
- javascript
