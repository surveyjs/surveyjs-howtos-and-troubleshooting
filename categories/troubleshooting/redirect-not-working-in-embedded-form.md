# Redirect to "Thank You" Page Not Working in Embedded SurveyJS Form

## Problem

I have added a URL in the "Thank You" page settings so that after a user submits the form, they are redirected to a custom "Thank You" page. This works as expected when the SurveyJS form runs standalone, but when it is embedded in another website using the `<embed>` HTML tag, the redirect never occurs.

## Solution

When you embed a SurveyJS form (or any web page) with the `<embed>` tag, it loads in its own isolated browsing context. Any navigation or redirect from within that embedded content happens only inside the embedded frame, without affecting the parent page. This behavior is enforced by browsers for security and compatibility reasons and cannot be overridden. To work around it, use one of the options below.

### Option 1: Let the parent page handle the redirect

Instead of relying on the "Thank You" page URL, handle the [`onComplete`](https://surveyjs.io/form-library/documentation/api-reference/survey-data-model#onComplete) event and have the parent page perform the redirect:

```js
// ...
// Omitted: `Survey.Model` creation
// ...
survey.onComplete.add(() => {
  window.top.location.href = "your-thank-you-page.html";
});
```

> This works only if the parent and embedded pages are on the **same origin** (same domain, protocol, and port). If they are on different origins, the browser will block access to `window.top`.

### Option 2: Use a `postMessage` for cross-origin communication

If the embedded survey and parent page are on **different origins**, use the `postMessage` API:

**In the embedded survey page:**

```js
survey.onComplete.add(() => {
  window.parent.postMessage({ action: "surveyCompleted" }, "*");
});
```

**In the parent page:**

```js
window.addEventListener("message", (event) => {
  if (event.data.action === "surveyCompleted") {
    window.location.href = "your-thank-you-page.html";
  }
});
```

### Option 3: Avoid `<embed>` for interactive redirects

If you control the host page, use an `<iframe>` instead of `<embed>`:

```html
<iframe src="your-survey-page.html"></iframe>
```

Alternatively, embed the survey directly in a container `<div>` on the host page using the SurveyJS API. This way, you can handle completion events in the same context and control navigation without cross-frame limitations.

<!-- ## Related Tags
- surveyjs
- embed-tag
- redirect
- thank-you-page
- iframe
- javascript -->