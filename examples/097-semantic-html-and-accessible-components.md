# React Example: A Semantic and Accessible Contact Page

This example creates a small React contact page using native HTML semantics. It includes page landmarks, a skip link, a reusable labeled input component, native form validation, a named radio group, a status message, and visible keyboard focus.

## File structure

    src/
    ├── ContactPage.tsx
    └── ContactPage.css

Render ContactPage from the application's entry point. The example uses React with TypeScript.

## 1. Build the page and form fields

Create src/ContactPage.tsx:

    import { useState, type FormEvent } from 'react';
    import './ContactPage.css';

    type TextFieldProps = {
      id: string;
      label: string;
      name: string;
      type?: 'text' | 'email';
      autoComplete: 'name' | 'email';
      description?: string;
      required?: boolean;
    };

    function TextField({
      id,
      label,
      name,
      type = 'text',
      autoComplete,
      description,
      required = false,
    }: TextFieldProps) {
      const descriptionId = id + '-description';

      return (
        <div className="field">
          <label className="field-label" htmlFor={id}>
            {label}
            {required ? <span> (required)</span> : null}
          </label>

          <input
            aria-describedby={description ? descriptionId : undefined}
            autoComplete={autoComplete}
            className="control"
            id={id}
            name={name}
            required={required}
            type={type}
          />

          {description ? (
            <p className="field-description" id={descriptionId}>
              {description}
            </p>
          ) : null}
        </div>
      );
    }

    export default function ContactPage() {
      const [submitted, setSubmitted] = useState(false);

      function handleSubmit(event: FormEvent<HTMLFormElement>) {
        event.preventDefault();

        // The browser blocks an invalid required or email field before this
        // handler runs because the form uses native constraint validation.
        event.currentTarget.reset();
        setSubmitted(true);
      }

      return (
        <>
          <a className="skip-link" href="#main-content">
            Skip to main content
          </a>

          <header className="site-header">
            <a className="brand" href="#top" id="top">
              Acme Support
            </a>

            <nav aria-label="Main navigation" className="site-nav">
              <a href="#contact">Contact</a>
              <a href="#help">Help</a>
            </nav>
          </header>

          <main className="page" id="main-content">
            <h1>Contact our support team</h1>
            <p className="intro">
              Send us a message and choose how you would like us to reply.
            </p>

            <section aria-labelledby="contact-heading" className="card" id="contact">
              <h2 id="contact-heading">Send a message</h2>
              <p id="required-note">Fields marked “required” must be completed.</p>

              <form
                aria-labelledby="contact-heading"
                className="contact-form"
                onSubmit={handleSubmit}
              >
                <TextField
                  autoComplete="name"
                  id="full-name"
                  label="Full name"
                  name="name"
                  required
                />

                <TextField
                  autoComplete="email"
                  description="We will use this address to reply to your message."
                  id="email-address"
                  label="Email address"
                  name="email"
                  required
                  type="email"
                />

                <div className="field">
                  <label className="field-label" htmlFor="message">
                    Message <span>(required)</span>
                  </label>
                  <textarea
                    aria-describedby="message-description"
                    className="control message-control"
                    id="message"
                    name="message"
                    required
                    rows={5}
                  />
                  <p className="field-description" id="message-description">
                    Include the details that will help us understand your question.
                  </p>
                </div>

                <fieldset className="reply-options">
                  <legend>Preferred reply method</legend>

                  <label className="choice">
                    <input
                      defaultChecked
                      name="replyMethod"
                      type="radio"
                      value="email"
                    />
                    <span>Email</span>
                  </label>

                  <label className="choice">
                    <input
                      name="replyMethod"
                      type="radio"
                      value="phone"
                    />
                    <span>Phone</span>
                  </label>
                </fieldset>

                <button className="submit-button" type="submit">
                  Send message
                </button>

                <p className="status-message" role="status">
                  {submitted ? 'Thanks. Your message is ready for our team.' : ''}
                </p>
              </form>
            </section>

            <section aria-labelledby="help-heading" className="help-section" id="help">
              <h2 id="help-heading">Before you send</h2>

              <details className="help-details">
                <summary>What happens after I send a message?</summary>
                <p>
                  Our support team reviews the message and replies using your
                  selected contact method.
                </p>
              </details>
            </section>
          </main>

          <footer className="site-footer">
            <p>Acme Support</p>
          </footer>
        </>
      );
    }

## 2. Add styles without removing focus indicators

Create src/ContactPage.css:

    :root {
      color: #172033;
      background: #f3f6fb;
      font-family: system-ui, sans-serif;
      font-synthesis: none;
      text-rendering: optimizeLegibility;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
    }

    a {
      color: #174dbb;
      text-underline-offset: 0.18em;
    }

    a:focus-visible,
    button:focus-visible,
    input:focus-visible,
    textarea:focus-visible,
    summary:focus-visible {
      outline: 3px solid #174dbb;
      outline-offset: 3px;
    }

    .skip-link {
      position: absolute;
      top: 0.75rem;
      left: 0.75rem;
      z-index: 1;
      padding: 0.75rem 1rem;
      transform: translateY(-180%);
      border-radius: 0.4rem;
      background: #ffffff;
      font-weight: 700;
    }

    .skip-link:focus {
      transform: translateY(0);
    }

    .site-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 1rem;
      padding: 1rem max(1rem, calc((100% - 58rem) / 2));
      background: #ffffff;
      border-bottom: 1px solid #cbd4e1;
    }

    .brand {
      color: #172033;
      font-size: 1.1rem;
      font-weight: 700;
      text-decoration: none;
    }

    .site-nav {
      display: flex;
      flex-wrap: wrap;
      gap: 1rem;
    }

    .page {
      width: min(100% - 2rem, 44rem);
      margin: 2.5rem auto;
    }

    h1,
    h2,
    p {
      margin-top: 0;
    }

    h1 {
      margin-bottom: 0.5rem;
      font-size: clamp(1.8rem, 5vw, 2.5rem);
      line-height: 1.15;
    }

    h2 {
      margin-bottom: 0.65rem;
      font-size: 1.4rem;
    }

    .intro {
      margin-bottom: 1.5rem;
      color: #43516a;
      line-height: 1.6;
    }

    .card {
      padding: clamp(1rem, 4vw, 1.5rem);
      border: 1px solid #cbd4e1;
      border-radius: 0.8rem;
      background: #ffffff;
      box-shadow: 0 0.5rem 1.5rem rgb(23 32 51 / 6%);
    }

    #required-note {
      color: #43516a;
      line-height: 1.5;
    }

    .contact-form {
      display: grid;
      gap: 1.25rem;
    }

    .field {
      display: grid;
      gap: 0.45rem;
    }

    .field-label,
    .reply-options legend {
      font-weight: 700;
    }

    .field-label span {
      color: #43516a;
      font-weight: 400;
    }

    .control {
      width: 100%;
      min-height: 2.8rem;
      padding: 0.65rem 0.75rem;
      border: 1px solid #76849a;
      border-radius: 0.4rem;
      background: #ffffff;
      color: #172033;
      font: inherit;
    }

    .message-control {
      min-height: 8rem;
      resize: vertical;
    }

    .field-description {
      margin-bottom: 0;
      color: #43516a;
      font-size: 0.95rem;
      line-height: 1.5;
    }

    .reply-options {
      display: flex;
      flex-wrap: wrap;
      gap: 0.8rem 1.25rem;
      margin: 0;
      padding: 1rem;
      border: 1px solid #cbd4e1;
      border-radius: 0.5rem;
    }

    .reply-options legend {
      padding: 0 0.25rem;
    }

    .choice {
      display: inline-flex;
      align-items: center;
      gap: 0.55rem;
      min-height: 2.75rem;
      cursor: pointer;
    }

    .choice input {
      width: 1.1rem;
      height: 1.1rem;
      accent-color: #174dbb;
    }

    .submit-button {
      justify-self: start;
      min-height: 2.9rem;
      padding: 0.65rem 1.1rem;
      border: 0;
      border-radius: 0.45rem;
      background: #174dbb;
      color: #ffffff;
      cursor: pointer;
      font: inherit;
      font-weight: 700;
    }

    .submit-button:hover {
      background: #103b95;
    }

    .status-message {
      min-height: 1.5rem;
      margin-bottom: 0;
      color: #17623b;
      line-height: 1.5;
    }

    .help-section {
      margin-top: 2rem;
    }

    .help-details {
      padding: 1rem;
      border: 1px solid #cbd4e1;
      border-radius: 0.5rem;
      background: #ffffff;
    }

    .help-details summary {
      cursor: pointer;
      font-weight: 700;
    }

    .help-details p {
      margin: 0.8rem 0 0;
      color: #43516a;
      line-height: 1.6;
    }

    .site-footer {
      padding: 1.25rem max(1rem, calc((100% - 58rem) / 2));
      border-top: 1px solid #cbd4e1;
      background: #ffffff;
      color: #43516a;
    }

    .site-footer p {
      margin: 0;
    }

## 3. What makes the markup meaningful?

- The skip link points to the main landmark, letting keyboard users bypass repeated navigation.
- header, nav, main, and footer identify page regions.
- Headings describe the page, the contact section, and its supporting help section.
- Navigation uses anchors with real in-page destinations.
- TextField renders an actual label and input, associates them with htmlFor and id, and connects optional help with aria-describedby.
- The textarea has its own visible label and instructions.
- fieldset and legend name a related radio group; each radio has a visible label.
- The submit control is a button with type="submit".
- details and summary provide a native disclosure interaction.
- The form uses browser constraint validation from required and type="email".
- The status region reports successful submission without taking focus away from the form.
- Focus styles remain visible for keyboard users.

The submit handler is only a front-end demonstration. A real product should send the data to its service, handle failures, and communicate the result.

## 4. Try it with a keyboard and assistive technology

1. Reload the page and press Tab. The skip link should become visible.
2. Activate the skip link and confirm focus/navigation reaches the main content.
3. Move through the navigation and form with Tab and Shift+Tab. Focus should remain visible.
4. Submit the empty form. The browser should identify a required field.
5. Enter an invalid email and submit again. The browser should reject the invalid format.
6. Use the arrow keys to move between the preferred reply radio options.
7. Expand the details element with its summary control.
8. Use a screen reader to navigate by landmarks, headings, links, and form controls.
9. Confirm the email instructions are associated with the email field.
10. Submit valid values and confirm the status message is announced.

## 5. Useful variations

- Add a server-side submission and display a clear failure message if it cannot be sent.
- Add field-level errors and connect each error to its field using aria-describedby.
- If custom validation is introduced, preserve native required and type semantics where useful, set invalid state accurately, and help users reach the first invalid field.
- Add an informative image with useful alt text, or use alt="" for an image that is purely decorative.
- Add an icon-only action only when it has an accessible name and an understandable focus style.
- Test at narrow widths, with enlarged text, and with browser zoom.

## Key lesson

Accessible React code is semantic HTML composed through components. Native elements supply useful roles and behavior; labels and descriptions supply context; ARIA fills specific gaps. Confirm the rendered result with keyboard and assistive technology instead of treating JSX or an automated scan as proof.
