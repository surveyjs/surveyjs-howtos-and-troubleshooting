# Display Only One Translation Tab with a Single Language in SurveyJS Creator

## Problem
You need to configure SurveyJS Creator to show only one translation tab for a specific language (e.g., Italian) in the user interface, hiding other tabs like Designer, Logic, Preview, and JSON Editor. Additionally, since multiple translation sessions in different languages may occur simultaneously, changes must be applied to the latest version of the survey JSON retrieved from the server to avoid conflicts.

## Solution
To display only one translation tab with a single language in SurveyJS Creator, disable all other tabs (Designer, Logic, Preview, JSON Editor) by setting their respective `show*Tab` properties to `false` and enable the translation tab with `showTranslationTab: true`. Use the `setVisibleLocales` method of the translation plugin to restrict the translation tab to a single language (e.g., `["it"]` for Italian). To handle concurrent translation sessions, fetch the latest survey JSON from the server before applying changes, update the translated strings, and save the updated JSON back to the server. This ensures translations are applied to the most recent survey version.

The following code example, based on a React and TypeScript implementation, demonstrates how to configure SurveyJS Creator to show a single language in the translation tab and manage server-side JSON updates.

## [Live Example](https://plnkr.co/edit/EdAKE6J0Svjxr1Ty)

## Code Example

### index.html
```html
<!DOCTYPE html>
<html>
<head>
  <title>SurveyJS Creator with Single Language Translation</title>
  <meta charset="utf-8" />
  <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <script src="https://unpkg.com/survey-core@1.11.6/survey.core.min.js"></script>
  <script src="https://unpkg.com/survey-creator-core@1.11.6/survey-creator-core.min.js"></script>
  <link href="https://unpkg.com/survey-core@1.11.6/defaultV2.min.css" rel="stylesheet" />
  <link href="https://unpkg.com/survey-creator-core@1.11.6/survey-creator-core.min.css" rel="stylesheet" />
</head>
<body>
  <div id="surveyCreatorContainer"></div>
  <script src="index.js"></script>
</body>
</html>
```

### index.js
```typescript
const localeToTranslate = "it";

const surveyJSON = {
  elements: [
    {
      type: "text",
      name: "question1",
      title: {
        default: "Enter your name",
        it: "Inserisci il tuo nome"
      }
    }
  ]
};

function loadJSONFromServer(creator, callback) {
  setTimeout(() => {
    creator.JSON = surveyJSON; // Simulate server fetch
    callback();
  }, 300);
}

function getTranslation(creator) {
  return creator.getPlugin("translation").model;
}

function setVisibleLocales(creator) {
  const translation = getTranslation(creator);
  translation.setVisibleLocales([localeToTranslate]);
}

function getTranslationObjectsByGroup(hash, group, isObj) {
  group.items.forEach(item => {
    if (item.isGroup) {
      getTranslationObjectsByGroup(hash, item, isObj);
    } else {
      const key = group.fullName + "." + item.name;
      hash[key] = isObj ? item : item.getLocText(localeToTranslate);
    }
  });
}

function getTranslatedObjects(creator, isObj) {
  const translation = getTranslation(creator);
  const hash = {};
  getTranslationObjectsByGroup(hash, translation.root, isObj);
  return hash;
}

function getTranslatedStrings(creator) {
  return getTranslatedObjects(creator, false);
}

function updateTranslatedStrings(creator, translatedHash) {
  const hash = getTranslatedObjects(creator, true);
  for (let key in hash) {
    const str = translatedHash[key];
    if (str) {
      hash[key].setLocText(localeToTranslate, str);
    }
  }
}

function SurveyCreatorRenderComponent() {
  const creator = new SurveyCreator.SurveyCreator({
    showDesignerTab: false,
    showLogicTab: false,
    showPreviewTab: false,
    showJSONEditorTab: false,
    showTranslationTab: true
  });

  // Add a save button to the translation tab toolbar
  creator.toolbar.actions.push(new Survey.Action({
    id: "svd-save-translation",
    iconName: "icon-save",
    iconSize: "auto",
    action: () => creator.saveSurvey(),
    active: false,
    locTitleName: "ed.saveSurvey",
    locTooltipName: "ed.saveSurveyTooltip",
    showTitle: false
  }));

  creator.saveSurveyFunc = (saveNo, callback) => {
    const translatedHash = getTranslatedStrings(creator);
    // Fetch the latest JSON and apply translation changes
    loadJSONFromServer(creator, () => {
      setVisibleLocales(creator);
      updateTranslatedStrings(creator, translatedHash);
      console.log('Updated Survey JSON:', creator.JSON); // Save to server here
      callback(saveNo, true);
    });
  };

  loadJSONFromServer(creator, () => {
    setVisibleLocales(creator);
  });

  return (<SurveyCreator.SurveyCreatorComponent creator={creator} />);
}

const root = ReactDOM.createRoot(document.getElementById("surveyCreatorContainer"));
root.render(<SurveyCreatorRenderComponent />);
```

### Notes
- **Dependencies**: Include `react`, `react-dom`, `survey-core`, and `survey-creator-core` with their CSS files:
  ```bash
  npm install react react-dom survey-core survey-creator-core
  ```
- **Single Language Translation**: The `setVisibleLocales([localeToTranslate])` method restricts the translation tab to a single language (e.g., Italian, `"it"`).
- **Server-Side JSON Handling**: The `loadJSONFromServer` function simulates fetching the latest survey JSON. In a real application, replace it with an API call (e.g., using `axios`) to fetch and save JSON to the server.
- **Translation Management**: The `getTranslatedStrings` and `updateTranslatedStrings` functions manage translation strings, ensuring changes are applied to the latest JSON version.
- **Save Button**: A custom save button is added to the toolbar to trigger `saveSurvey`, which fetches the latest JSON, applies translations, and saves the result.
- **Plunker Reference**: For a working example, see the [Plunker example](https://plnkr.co/edit/EdAKE6J0Svjxr1Ty).
- **Customization**: Adjust `localeToTranslate` to the desired language code. Add error handling or validation in `loadJSONFromServer` for robust server communication.
- **Concurrent Sessions**: Fetching the latest JSON before saving ensures translations are applied to the most recent survey version, reducing conflicts.