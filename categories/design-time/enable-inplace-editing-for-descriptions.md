# Enable In-Place Editing for Element Descriptions

## Problem

By default, survey authors can edit element *titles* directly on the design surface, but element *descriptions* do not support in-place editing.

## Solution

To enable in-place editing for element descriptions, use the `Serializer` API to define a placeholder for the `description` property of a specific element type (for example, `question`). This enables authors to edit descriptions directly on the design surface without using the Property Grid.

### Code Sample

```javascript
import { Serializer } from "survey-core";

Serializer.getProperty("question", "description").placeholder = "Description";
```