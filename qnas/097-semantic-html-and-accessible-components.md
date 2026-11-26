# Semantic HTML and Accessible React Components: Questions and Answers

## Foundations

### 1. Does React make a page accessible automatically?

No. React renders elements and updates them from state and props. The rendered HTML still needs appropriate semantics, names, relationships, and usable behavior.

### 2. What does semantic HTML mean?

It means choosing HTML elements according to their purpose. For example, use a button for an action, an anchor for navigation, and a label for a form control.

### 3. Why is semantic HTML useful?

Browsers and assistive technologies can expose the element's purpose and built-in behavior. Semantic markup also helps keyboard users and makes the document easier to inspect and maintain.

### 4. Should I choose an element by its default appearance?

No. Choose it by function, then style it with CSS. A heading should be a heading because it labels a section, not because its browser font is large.

### 5. Is a div always inaccessible?

No. A div is appropriate as a generic container when no more specific element fits. It becomes a problem when it is used in place of an interactive or structural element with useful native behavior.

## Structure and headings

### 6. What are landmarks?

Landmarks identify major page regions such as navigation, main content, and complementary information. Elements such as header, nav, main, aside, and footer often provide these semantics.

### 7. Should every wrapper be a section?

No. Use section for a thematic group, usually with a heading. Generic layout wrappers can remain div elements. Too many region-like sections can make assistive technology navigation noisy.

### 8. How should heading levels be chosen?

Choose levels based on the document hierarchy. A typical page has a clear h1 title, h2 section headings, and h3 headings for subsections. Do not select a heading just to get a particular font size.

### 9. Must a page contain exactly one h1?

HTML allows more than one h1, but a clear page title and logical heading hierarchy usually make the page easier to navigate. Use headings to describe the content structure rather than following a count as a styling rule.

### 10. When should I use article?

Use article for content that can stand on its own, such as a post or news item. Use section to group related content within a larger page or article.

### 11. Do multiple navigation landmarks need names?

Give multiple nav regions distinguishable names, such as "Main navigation" and "Help sections," so users can tell them apart.

### 12. Should a section have a heading?

Usually, yes. A heading gives the section a meaningful label. If a section is only a styling wrapper, a div may be more appropriate.

## Links, buttons, and controls

### 13. What is the difference between an anchor and a button?

An anchor navigates to a destination. A button performs an action, such as opening a panel or submitting a form.

### 14. Why should I avoid href="#" for an action?

It presents the control as a link without a meaningful destination and can cause unexpected navigation or page movement. Use a button for an action.

### 15. Why is a clickable div a poor button replacement?

It does not automatically have a button role, keyboard activation, focus behavior, or native control semantics. A real button provides these behaviors by default.

### 16. Which button type should I use?

Use type="submit" for the control that submits a form. Use type="button" for other actions inside the form, so they do not accidentally submit it.

### 17. What makes link text useful?

The purpose should be understandable from the link text itself or its clear surrounding context. "View billing history" is more specific than "Click here" or repeated "Read more" labels.

### 18. Does visible text usually provide a button's accessible name?

Yes. Text such as "Save draft" normally provides the button's name. For an icon-only button, provide a suitable name, such as with aria-label.

### 19. Should a decorative icon inside a labeled button be announced?

Usually not. Mark the decorative icon aria-hidden="true" while keeping the button's visible text available as the control's name. Do not hide an icon if it is the only source of important information.

## Forms and accessible names

### 20. How do I associate a label with an input in JSX?

Give the input an id and set the label's htmlFor to the same value. The input should also have a name when its value is submitted as form data.

### 21. Can a label wrap its input instead?

Yes. A label may contain its associated control. An explicit htmlFor and id relationship is also valid and is often easier to inspect in a component.

### 22. Is placeholder text a substitute for a label?

No. Placeholder text can disappear when the user enters a value and may be visually faint. Provide a persistent label; use placeholder only for a helpful example or hint.

### 23. What are fieldset and legend for?

Use fieldset to group related form controls, and legend to name the group. A set of radio choices such as a preferred contact method is a common example.

### 24. Should I add aria-required to an input with required?

Usually no. The native required attribute already communicates that constraint. Avoid duplicate or contradictory ARIA where native HTML already provides the meaning.

### 25. What is the difference between an accessible name and description?

The name identifies a control, for example "Email address." A description provides supporting context, for example "Use the address associated with your account."

### 26. When should aria-describedby be used?

Use it to associate existing instructions or supporting text with a control. Its value references the id of the descriptive element or elements.

### 27. When should aria-labelledby be used?

Use it when existing visible text should name a control or region. It references one or more element ids whose text provides the name.

### 28. When should aria-label be used?

Use aria-label when the element needs an accessible name but there is no suitable visible text to reference or use. Prefer visible labels when practical, and keep the ARIA name consistent with visible wording.

## Images and ARIA

### 29. How should an informative image be labeled?

Provide alt text that communicates the information the image contributes in its context. Keep it relevant; a long visual inventory is not automatically a better alternative.

### 30. What alt value should a decorative image use?

Use an empty alt value, alt="", when the image adds no information and should be skipped by assistive technology. Do not omit alt on an image element.

### 31. What does aria-hidden="true" do?

It removes an element and its descendants from the accessibility tree. Use it only for non-interactive, nonessential decoration; do not hide useful content or an operable control.

### 32. Does role="button" make a div behave like a button?

No. A role communicates a semantic role, but it does not add focusability, keyboard activation, disabled behavior, or other native button behavior.

### 33. What does "prefer native HTML over ARIA" mean?

If a native element already expresses the meaning and behavior, use it. ARIA can supplement a native element when necessary but should not replace a suitable button, link, label, or form control.

### 34. Can adding more ARIA improve accessibility?

Not necessarily. Incorrect or unnecessary roles and states can conflict with what native HTML already exposes. Add only the information and behavior the interface actually needs.

## Reusable React components

### 35. What should a reusable input component accept?

It should expose the information needed to render a proper control, such as a unique id, visible label, name, type, required state, and possibly help text. It should associate the label with the input it renders.

### 36. Why should input ids be unique?

A label's htmlFor relationship targets an id. Duplicate ids can associate labels with the wrong control and make the document invalid or confusing.

### 37. Should a component forward accessibility props?

If callers need to supply an accessible name, description, focus behavior, or event handling, pass those props to the correct native element. Do not drop or apply them to an unrelated wrapper.

### 38. Is it useful to make every control a polymorphic component?

Not always. An abstraction that can unpredictably render a button, link, or div can hide important semantics. Prefer clear components whose purpose and required props are obvious.

### 39. Can a visually hidden label be accessible?

Yes, if it remains available in the accessibility tree and is programmatically associated with the control. A visible label is better when the design allows it.

## Review and common mistakes

### 40. What should I inspect in the browser's accessibility tree?

Check the rendered roles, accessible names, values, states, and relationships. The JSX source alone does not prove that a component produced the intended result.

### 41. What keyboard checks should I perform?

Navigate through interactive controls with the keyboard, confirm that focus is visible and ordered sensibly, and activate links and buttons using their expected keys. Ensure custom behavior does not trap or lose focus.

### 42. Does an automated accessibility scan prove a page is accessible?

No. Automated tools can detect some structural and contrast issues, but they cannot decide whether text is clear, an image alternative is useful, or a workflow makes sense. Combine automation with keyboard, screen reader, and human review.

### 43. Why can a passing accessibility scan miss a bad link?

A scan may verify that a link has text but cannot determine whether "More" or "Click here" explains its destination to the people using it. Review meaning in context and when links are read out of context.

### 44. What are common accessible React mistakes?

Common mistakes include clickable divs, placeholder-only form controls, misleading link text, skipped heading structure, unnecessary ARIA, decorative images with verbose alt text, and custom components that discard labels or focus props.

### 45. What is the main rule for an accessible React component?

Render the native HTML element that fits the purpose, preserve its built-in behavior, provide clear names and relationships, and verify the actual browser output with keyboard and assistive technology.

## Quick review

- Navigation means anchor; action means button.
- Use headings and landmarks to communicate document structure.
- Associate every form control with a useful label.
- Write alt text according to the image's purpose.
- Use ARIA to fill genuine semantic gaps, not to imitate native behavior.
- Inspect and test the rendered interface, not only the JSX.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="096-react-native-and-shared-react-concepts.md">096 - React Native and the Shared React Mental Model: Questions and Answers</a></td><td><a href="../lessons/097-semantic-html-and-accessible-components.md">Lesson</a></td><td><a href="../examples/097-semantic-html-and-accessible-components.md">Example</a></td><td><a href="098-keyboard-focus-and-dialogs.md">098 - Keyboard Support, Focus, and Dialogs: Questions and Answers</a></td></tr>
  </tbody>
</table>
