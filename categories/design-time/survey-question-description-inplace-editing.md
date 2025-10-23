# Enable In-Place Editing for Question Description

## Problem
Content editors can create surveys and edit page descriptions directly within a design surface. However, inline editors for question descriptions are not available.

## Solution
To enable in-place editors (placeholders) for question descriptions in SurveyJS, configure a description placeholder for the question object using the `Serializer` from `survey-core`. This allows users to edit descriptions directly without navigating to the properties tab.

### Code Sample
```javascript
import { Serializer } from "survey-core";

Serializer.getProperty('question', 'description').placeholder = "Description";
```