# Show Only the Translation Tab with a Single Language in SurveyJS Creator

## Problem

I want to configure SurveyJS Creator to display only the Translation tab for a specific language (e.g., Italian), while hiding other tabs (Designer, Logic, Preview, and JSON Editor). Since multiple translators may work on different languages at the same time, any changes must always be applied to the latest survey JSON schema fetched from the server to prevent conflicts.

## Solution

To configure SurveyJS Creator for this scenario, follow the steps below:

1. **Enable only the Translation tab**  
  Set all [`show_TabName_Tab`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#showDesignerTab) properties to `false`, except for [`showTranslationTab`](https://surveyjs.io/survey-creator/documentation/api-reference/survey-creator#showTranslationTab), which should be `true`.

1. **Limit available languages to one**  
  To prevent adding unsupported locales, assign the allowed codes to `surveyLocalization.supportedLocales`. Then, limit visible locales in the Translation tab by calling its `setVisibleLocales` method (for example, `[ "it" ]` for Italian).

1. **Handle concurrent sessions safely**  
  Always fetch the most recent survey JSON schema from the server before applying translations. Update the translated strings in that version, then save the updated JSON back to the server. See the `saveSurveyFunc` implementation below.

### Code Example

```js
import { SurveyCreatorModel } from "survey-creator-core";
import { Action, surveyLocalization } from "survey-core";

const targetLocale = "it";

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

// Step 1: Enable only the Translation tab
const creator = new SurveyCreatorModel({
  showDesignerTab: false,
  showLogicTab: false,
  showPreviewTab: false,
  showJSONEditorTab: false,
  showTranslationTab: true
});

function loadJSONFromServer(creator, callback) {
  setTimeout(() => {
    creator.JSON = surveyJSON; // Simulate server fetch
    callback();
  }, 300);
}

function getTranslationTab(creator) {
  return creator.getPlugin("translation").model;
}

// Step 2: Limit available languages to one
surveyLocalization.supportedLocales = [targetLocale];
function setVisibleLocales(creator) {
  const translation = getTranslationTab(creator);
  translation.setVisibleLocales([targetLocale]);
}

function getTranslationObjectsByGroup(hash, group, isObj) {
  group.items.forEach(item => {
    if (item.isGroup) {
      getTranslationObjectsByGroup(hash, item, isObj);
    } else {
      const key = group.fullName + "." + item.name;
      hash[key] = isObj ? item : item.getLocText(targetLocale);
    }
  });
}

function getTranslatedObjects(creator, isObj) {
  const translation = getTranslationTab(creator);
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
      hash[key].setLocText(targetLocale, str);
    }
  }
}

// A toolbar button that saves the survey JSON
creator.toolbar.actions.push(new Action({
  id: "svd-save-translation",
  iconName: "icon-save",
  iconSize: "auto",
  action: () => creator.saveSurvey(),
  active: false,
  locTitleName: "ed.saveSurvey",
  locTooltipName: "ed.saveSurveyTooltip",
  showTitle: false
}));

// Step 3: Handle concurrent sessions safely
creator.saveSurveyFunc = (saveNo, callback) => {
  const translatedHash = getTranslatedStrings(creator);
  loadJSONFromServer(creator, () => {
    setVisibleLocales(creator);
    updateTranslatedStrings(creator, translatedHash);
    console.log('Updated Survey JSON:', creator.JSON); // Save `creator.JSON` to server here
    callback(saveNo, true);
  });
};

loadJSONFromServer(creator, () => {
  setVisibleLocales(creator);
});
```

[View Live Example](https://plnkr.co/edit/MN3X8OMH13NmrU30)