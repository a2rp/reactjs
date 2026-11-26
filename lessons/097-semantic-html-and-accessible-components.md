# Semantic HTML and Accessible React Components

## Learning goals

After this lesson, you should be able to:

- Explain why semantic HTML is the foundation of an accessible React interface.
- Choose links, buttons, landmarks, headings, lists, and form controls by purpose.
- Give inputs and icon controls a programmatic name.
- Use ARIA to add information without replacing native behavior.
- Build reusable React components without hiding the semantics consumers need.
- Check rendered markup, keyboard behavior, and the accessibility tree.

## 1. React renders HTML, so HTML semantics still matter

A React component ultimately renders browser elements. React does not make an element accessible simply because it is written as a component or has an event handler.

Semantic HTML communicates an element's purpose and relationships to browsers and assistive technology. A native button already exposes its button role, supports keyboard activation, and participates in focus. A div with an onClick handler does not automatically acquire those behaviors.

Start with the HTML element that matches the task. Add React state and event handling to that element. Add ARIA only when native HTML does not express a needed name, state, or relationship.

## 2. Structure pages with landmarks and headings

Use page landmarks to make major regions easier to identify and navigate:

- header for introductory content or site-level material.
- nav for a group of navigation links.
- main for the page's primary content.
- aside for related content.
- footer for footer content.
- section for a labeled thematic group.
- article for content that can stand on its own.

A typical page may look like this:

    function HelpPage() {
      return (
        <>
          <header>
            <a href="/">Acme Help</a>
          </header>

          <nav aria-label="Help sections">
            <a href="/guides">Guides</a>
            <a href="/contact">Contact</a>
          </nav>

          <main>
            <h1>Help center</h1>
            <section aria-labelledby="popular-guides">
              <h2 id="popular-guides">Popular guides</h2>
              <ul>
                <li><a href="/guides/account">Manage your account</a></li>
                <li><a href="/guides/billing">Understand billing</a></li>
              </ul>
            </section>
          </main>

          <footer>Support hours: Monday to Friday</footer>
        </>
      );
    }

Use heading levels to describe the structure: a page title commonly begins with h1, major sections with h2, and nested sections with h3. Do not choose a heading level just because its default font size looks right. Use CSS for appearance.

A section should normally have a heading that identifies it. Avoid adding landmark elements to every wrapper: excessive regions make navigation noisier. If a page has multiple nav landmarks, give each one a distinguishable accessible name.

## 3. Choose links and buttons by what happens

An anchor navigates to a destination. A button performs an action in the current interface.

    <a href="/account/settings">Account settings</a>

    <button type="button" onClick={openSettings}>
      Open settings panel
    </button>

Avoid anchors with href="#" or javascript URLs when the intent is an action. Avoid clickable div or span elements. Native anchors and buttons provide expected keyboard behavior and expose their purpose to assistive technology.

Use a real destination in an anchor. Use type="button" for an action button inside a form that should not submit it. Use type="submit" for the form's submit control. Give controls visible text that makes sense when read on its own; "Save changes" is more useful than "Click here."

## 4. Label form controls

Every form control needs a name that tells the user what information it expects. The visible label is usually the clearest choice:

    <label htmlFor="email">Email address</label>
    <input
      id="email"
      name="email"
      type="email"
      autoComplete="email"
      required
    />

In JSX, use htmlFor to associate a label with an element's id. A label can also wrap its control, but an explicit htmlFor/id association is easy to inspect and works well in reusable form layouts.

A placeholder is a hint, not a replacement for a persistent label. Placeholder text can disappear when a value is entered and may be hard to read. Required status, format instructions, and examples should be visible and associated with the control.

For related controls, group them with fieldset and legend:

    <fieldset>
      <legend>Preferred reply method</legend>
      <label>
        <input type="radio" name="reply" value="email" />
        Email
      </label>
      <label>
        <input type="radio" name="reply" value="phone" />
        Phone
      </label>
    </fieldset>

Use native input types such as email, checkbox, radio, and date where they fit. They communicate useful information and provide platform behavior.

## 5. Give images text alternatives based on their purpose

For an informative image, write alt text that conveys the information the image contributes. Do not describe every visual detail if only one fact matters.

    <img src="/images/team.webp" alt="The support team meeting at a table" />

For a purely decorative image that adds no information, use an empty alt value so assistive technology can skip it:

    <img src="/images/blue-divider.svg" alt="" />

An icon inside a button often decorates visible button text. Hide that icon from assistive technology and keep the button's text as its name:

    <button type="button" onClick={saveDraft}>
      <SaveIcon aria-hidden="true" />
      Save draft
    </button>

If an image or icon is the only content of a control, the control still needs an accessible name, for example an aria-label on the button. The name should describe the action, not the icon's shape.

## 6. Understand accessible names and descriptions

An accessible name identifies a control. A visible label, button text, or link text usually provides it. An accessible description adds supporting information such as a format instruction or error.

Use aria-label when a visible label is not available and the element needs a name. Prefer visible text where possible, since it helps everyone. Use aria-labelledby to point to existing visible text when that text should name a control or region. Use aria-describedby to connect instructions or supporting text.

    <label htmlFor="postal-code">Postal code</label>
    <input
      id="postal-code"
      aria-describedby="postal-help"
      name="postalCode"
    />
    <p id="postal-help">Enter the code for your delivery address.</p>

Do not add an aria-label that conflicts with visible wording. Some assistive technology experiences use the accessible name instead of the visible text, so matching those names helps people use speech input as well.

## 7. Prefer native HTML over unnecessary ARIA

ARIA adds accessibility information; it does not implement behavior. A role does not add keyboard handling, focus management, form submission, or browser defaults.

Prefer:

    <button type="button" onClick={toggleDetails}>
      {expanded ? 'Hide details' : 'Show details'}
    </button>

Avoid recreating the same control as a div with role="button". If a custom widget is truly required, its author must supply the correct role, accessible name, state, focus behavior, and expected keyboard interaction.

Do not add roles that repeat the native element's meaning without a reason. Incorrect or contradictory ARIA can make a page harder to understand than HTML alone. Hide decorative content with aria-hidden="true" only when hiding it will not remove information or an interactive control from assistive technology.

## 8. Build reusable components that preserve semantics

A reusable component should make correct markup easy and should allow callers to provide the information its HTML element needs.

    type TextFieldProps = {
      id: string;
      label: string;
      name: string;
      type?: 'text' | 'email' | 'search';
      autoComplete?: string;
      required?: boolean;
    };

    function TextField({
      id,
      label,
      name,
      type = 'text',
      autoComplete,
      required = false,
    }: TextFieldProps) {
      return (
        <div className="field">
          <label htmlFor={id}>
            {label}
            {required ? ' (required)' : ''}
          </label>
          <input
            autoComplete={autoComplete}
            id={id}
            name={name}
            required={required}
            type={type}
          />
        </div>
      );
    }

The component owns the label-to-control relationship, and the caller supplies a stable unique id. If a component accepts ARIA attributes, event handlers, or focus props from its caller, pass them to the correct native element. Avoid abstractions that hide whether the component renders a link, button, or form control.

If a component renders an interactive element, document its semantics and required props. Do not silently change an anchor into a button or a button into a generic container based on styling.

## 9. React updates do not replace semantic structure

When state changes, React updates the rendered tree. The updated tree still needs correct HTML structure and control names.

For a form, keep labels connected to controls and use buttons for actions. For a navigation menu, use links for destinations. For a data collection, use list markup when the content is a list. For a heading, render an actual h1-h6 element instead of styling a generic paragraph to look like one.

A visually hidden label can be useful when the design cannot show a visible label, but it should still be rendered and programmatically associated. Hiding all labels and relying on placeholders weakens the interface.

## 10. Accessibility is more than markup

Semantic markup gives browsers and assistive technology a useful foundation. It does not guarantee that every interaction is usable.

Also consider:

- Visible keyboard focus and logical focus movement.
- Clear labels and instructions.
- Text and non-text contrast.
- Touch and pointer target size.
- Validation feedback and how users recover from errors.
- Dynamic updates and whether users can notice them.
- Zoom, reflow, and different input methods.
- Testing with keyboard and assistive technology.

These topics build on semantics and are covered in more detail in the surrounding lessons.

## 11. A practical review process

Review the rendered page rather than only the JSX source:

1. Inspect the browser's accessibility tree to confirm roles, names, values, and relationships.
2. Navigate using only the keyboard. Check that links and buttons are reachable and that focus is visible.
3. Activate each control using its expected keyboard input.
4. Check labels and instructions without relying on placeholder text.
5. Use a screen reader to navigate headings, landmarks, links, and form controls.
6. Check that the content still makes sense when CSS is disabled or text is enlarged.
7. Test the error and success states as well as the initial view.

Automated tools can find some problems, but they cannot determine whether a label is meaningful or whether a workflow is understandable. Combine automated checks with human review.

## Common mistakes

- Using div or span for a control that should be a button, link, or input.
- Choosing a semantic element only for its default appearance.
- Skipping heading levels to make text look smaller or larger.
- Using a placeholder instead of a label.
- Giving several links the same vague text such as "Read more."
- Using an anchor for an action or a button for navigation.
- Adding ARIA roles without implementing the required behavior.
- Hiding an informative image with an empty alt value.
- Writing alt text for a decorative image that creates unnecessary noise.
- Building a custom input without preserving its label, name, and focus behavior.
- Assuming a passing automated scan proves the interface is accessible.

## Summary

In React, semantic HTML is still the foundation of accessible interfaces. Choose elements by purpose, provide useful names and relationships, and let native browser behavior do its work. Use ARIA to fill genuine gaps, not to replace elements that already express the right meaning. Reusable components should preserve the semantics and behavior of the native elements they render.

## Practice checklist

1. Replace a clickable div with the correct link or button.
2. Add a heading and landmark structure to a page.
3. Connect every form input to a visible label.
4. Decide whether each image is informative or decorative and choose alt text accordingly.
5. Inspect a component's rendered accessibility tree and use it with only a keyboard.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="096-react-native-and-shared-react-concepts.md">096 - React Native and the Shared React Mental Model</a></td><td><a href="../qnas/097-semantic-html-and-accessible-components.md">Q&amp;A</a></td><td><a href="../examples/097-semantic-html-and-accessible-components.md">Example</a></td><td><a href="098-keyboard-focus-and-dialogs.md">098 - Keyboard Support, Focus, and Dialogs in React</a></td></tr>
  </tbody>
</table>
