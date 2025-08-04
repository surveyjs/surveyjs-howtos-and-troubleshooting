# Redirection to Thank You Page Not Working in Embedded SurveyJS Form

## Question
I have added a URL in the Thank You page settings so that when a user submits the SurveyJS form, it redirects to a Thank You page. This works fine standalone, but when I embed the SurveyJS form in another website using the `<embed>` HTML tag, the redirection does not occur. How can I fix this?

## Answer
When embedding a SurveyJS form (or any web content) using the `<embed>` HTML tag, redirections initiated from within the embedded content are confined to the embed itself and cannot affect or navigate the parent page. This is due to how the `<embed>` element isolates content for security and compatibility reasons—it creates a separate browsing context similar to an `<iframe>`, where navigation (including redirects) does not propagate to the top-level window.

### Why Redirects Don't Work in `<embed>`
- The `<embed>` tag is primarily for embedding external plugins or content (e.g., PDFs, multimedia), but when used for HTML, it behaves like an isolated frame. Any script or form submission attempting a redirect (e.g., `window.location = "thankyou.html"`) stays within the embed's context.
- Security policies, such as the same-origin policy and content isolation, prevent embedded content from controlling the parent page to avoid risks like unauthorized navigation or cross-site scripting (XSS).
- This issue is common in survey tools; redirects "trap" inside the embed, showing the Thank You page only within the small embed area instead of redirecting the full page.

For SurveyJS specifically, if you're embedding via `<embed src="survey.html">`, the form's completion redirect (set in the Thank You page URL) won't break out. Instead, consider proper embedding methods:
- Embed SurveyJS directly into a `<div>` using JavaScript (recommended for full control).
- Use an `<iframe>` with the `sandbox` attribute allowing top navigation (e.g., `sandbox="allow-top-navigation"`), but test for security implications.
- Handle redirection manually in the parent page using SurveyJS events like `onComplete`.

### Alternative: Handle Redirect with SurveyJS `onComplete` Event
To redirect the parent page on form submission, embed SurveyJS in a `<div>` and use the `onComplete` event in your JavaScript:

```javascript
import { Model } from 'survey-core';
import { Survey } from 'survey-react-ui'; // Or appropriate library for your framework

const surveyJson = { /* Your survey JSON with Thank You page URL */ };
const survey = new Model(surveyJson);

survey.onComplete.add((sender) => {
  window.location.href = 'thankyou.html'; // Redirect parent page
});

// Render in a div: <div id="surveyElement"></div>
new Survey(survey).render('surveyElement');
```

This ensures the redirect happens in the parent context. If using `<iframe>`, add `sandbox="allow-top-navigation allow-scripts"` to the `<iframe>` tag to permit it.

**Pros**: Secure and reliable; works with SurveyJS events.  
**Cons**: Requires switching from `<embed>` to proper JS embedding or `<iframe>`; may need CORS adjustments for cross-origin embeds.

## Recommended Approach
Avoid `<embed>` for interactive forms—use SurveyJS's JavaScript embedding in a `<div>` and handle redirects via `onComplete`. This is the most flexible and secure method for SurveyJS integrations.

## Related Tags
- surveyjs
- embed-tag
- redirect
- thank-you-page
- iframe
- javascript

## References
- [MDN: `<embed>` Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/embed)<grok:render card_id="36e658" card_type="citation_card" type="render_inline_citation">
<argument name="citation_id">20</argument>
</grok:render> – Details on `<embed>` isolation and limited scripting.
- [MDN: `<iframe>` Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)<grok:render card_id="4a677f" card_type="citation_card" type="render_inline_citation">
<argument name="citation_id">31</argument>
