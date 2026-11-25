# Design Systems, Shared Components, and Component Libraries — Examples

This example creates a small application-level UI system: shared design tokens, a button, a controlled text field, and a card. A profile settings feature composes these components.

This is an in-repository shared UI area, not a published package. There is only one feature using these components, so the example focuses on a clear local API rather than package build configuration.

## 1. File structure

```text
src/
  main.jsx
  app/
    App.jsx
    app.css
    tokens.css
  features/
    profile/
      ProfileSettings.jsx
      profile-settings.css
  shared/
    ui/
      Button.jsx
      button.css
      Card.jsx
      card.css
      TextField.jsx
      text-field.css
```

## 2. Define shared design tokens

Create `src/app/tokens.css`:

```css
:root {
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
    "Segoe UI", sans-serif;
  color: #192235;
  background: #f3f5fa;
  font-synthesis: none;
  text-rendering: optimizeLegibility;

  --color-text: #192235;
  --color-muted: #626d81;
  --color-surface: #ffffff;
  --color-primary: #3448b5;
  --color-primary-hover: #283a9c;
  --color-border: #dce1ec;
  --color-danger: #a32637;
  --color-danger-surface: #fff1f2;
  --color-focus: #8897ed;

  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-6: 1.5rem;

  --radius-control: 0.5rem;
  --radius-card: 1rem;
}
```

The tokens have semantic names so multiple components can use the same design decisions. They are a small starting set, not a requirement to create a token for every CSS value.

## 3. Create a button component

Create `src/shared/ui/Button.jsx`:

```jsx
import "./button.css";

const variantClasses = {
  primary: "ui-button--primary",
  secondary: "ui-button--secondary",
  danger: "ui-button--danger",
};

const sizeClasses = {
  small: "ui-button--small",
  medium: "ui-button--medium",
};

export default function Button({
  variant = "primary",
  size = "medium",
  type = "button",
  disabled = false,
  className = "",
  children,
  ...nativeProps
}) {
  const variantClass =
    variantClasses[variant] ?? variantClasses.primary;
  const sizeClass = sizeClasses[size] ?? sizeClasses.medium;
  const classes = [
    "ui-button",
    variantClass,
    sizeClass,
    className,
  ]
    .filter(Boolean)
    .join(" ");

  return (
    <button
      {...nativeProps}
      type={type}
      disabled={disabled}
      className={classes}
    >
      {children}
    </button>
  );
}
```

Create `src/shared/ui/button.css`:

```css
.ui-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: 1px solid transparent;
  border-radius: var(--radius-control);
  font-weight: 700;
  line-height: 1.2;
  transition:
    background-color 140ms ease,
    border-color 140ms ease,
    color 140ms ease;
}

.ui-button--small {
  min-height: 2.25rem;
  padding: var(--space-2) var(--space-3);
  font-size: 0.85rem;
}

.ui-button--medium {
  min-height: 2.75rem;
  padding: var(--space-2) var(--space-4);
  font-size: 0.95rem;
}

.ui-button--primary {
  background: var(--color-primary);
  color: var(--color-surface);
}

.ui-button--primary:hover:not(:disabled) {
  background: var(--color-primary-hover);
}

.ui-button--secondary {
  border-color: var(--color-border);
  background: var(--color-surface);
  color: var(--color-text);
}

.ui-button--danger {
  background: var(--color-danger);
  color: var(--color-surface);
}

.ui-button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.ui-button:focus-visible {
  outline: 3px solid var(--color-focus);
  outline-offset: 2px;
}
```

The component renders a native button, forwards useful native props such as `onClick` and `aria-label`, and exposes a small set of variants and sizes. It does not pretend to be a link or replace the browser's button behavior.

## 4. Create a controlled text field

Create `src/shared/ui/TextField.jsx`:

```jsx
import "./text-field.css";

export default function TextField({
  id,
  label,
  hint,
  error,
  value = "",
  onChange,
  type = "text",
  className = "",
  "aria-describedby": describedBy,
  ...inputProps
}) {
  const descriptionIds = [
    describedBy,
    hint ? `${id}-hint` : "",
    error ? `${id}-error` : "",
  ]
    .filter(Boolean)
    .join(" ");

  return (
    <div className={["ui-field", className].filter(Boolean).join(" ")}>
      <label className="ui-field__label" htmlFor={id}>
        {label}
      </label>

      <input
        {...inputProps}
        id={id}
        type={type}
        value={value}
        onChange={onChange}
        aria-invalid={Boolean(error)}
        aria-describedby={descriptionIds || undefined}
        className="ui-field__input"
      />

      {hint && (
        <p className="ui-field__hint" id={`${id}-hint`}>
          {hint}
        </p>
      )}

      {error && (
        <p className="ui-field__error" id={`${id}-error`}>
          {error}
        </p>
      )}
    </div>
  );
}
```

Create `src/shared/ui/text-field.css`:

```css
.ui-field {
  display: grid;
  gap: var(--space-2);
}

.ui-field__label {
  color: var(--color-text);
  font-size: 0.9rem;
  font-weight: 650;
}

.ui-field__input {
  width: 100%;
  min-height: 2.75rem;
  border: 1px solid #bac3d4;
  border-radius: var(--radius-control);
  padding: var(--space-2) var(--space-3);
  background: var(--color-surface);
  color: var(--color-text);
}

.ui-field__input[aria-invalid="true"] {
  border-color: var(--color-danger);
}

.ui-field__input:focus-visible {
  outline: 3px solid var(--color-focus);
  outline-offset: 2px;
}

.ui-field__hint,
.ui-field__error {
  margin: 0;
  font-size: 0.85rem;
  line-height: 1.45;
}

.ui-field__hint {
  color: var(--color-muted);
}

.ui-field__error {
  color: var(--color-danger);
}
```

The field is controlled: its caller provides `value` and `onChange`. The required `id` connects the visible label to the input. Hint and error messages are connected as descriptions when present. Other native input props such as `name`, `autoComplete`, and `required` are forwarded.

## 5. Create a composable card

Create `src/shared/ui/Card.jsx`:

```jsx
import "./card.css";

export default function Card({ title, description, children, className = "" }) {
  const classes = ["ui-card", className].filter(Boolean).join(" ");

  return (
    <article className={classes}>
      <header className="ui-card__header">
        <h2 className="ui-card__title">{title}</h2>
        {description && (
          <p className="ui-card__description">{description}</p>
        )}
      </header>

      <div className="ui-card__body">{children}</div>
    </article>
  );
}
```

Create `src/shared/ui/card.css`:

```css
.ui-card {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-card);
  padding: var(--space-6);
  background: var(--color-surface);
  box-shadow: 0 10px 30px rgb(25 39 74 / 6%);
}

.ui-card__header {
  margin-bottom: var(--space-6);
}

.ui-card__title {
  margin: 0 0 var(--space-2);
  color: var(--color-text);
  font-size: 1.25rem;
}

.ui-card__description {
  margin: 0;
  color: var(--color-muted);
  line-height: 1.55;
}
```

The card provides structure and accepts its body through `children`. The caller chooses the contents without the card needing to know about profile settings.

## 6. Compose the components in a feature

Create `src/features/profile/ProfileSettings.jsx`:

```jsx
import { useState } from "react";
import Button from "../../shared/ui/Button.jsx";
import Card from "../../shared/ui/Card.jsx";
import TextField from "../../shared/ui/TextField.jsx";
import "./profile-settings.css";

export default function ProfileSettings() {
  const [savedName, setSavedName] = useState("Jordan Lee");
  const [displayName, setDisplayName] = useState("Jordan Lee");
  const [showError, setShowError] = useState(false);
  const [statusMessage, setStatusMessage] = useState("");

  const normalizedName = displayName.trim();
  const hasChanges = displayName !== savedName;
  const error =
    showError && normalizedName.length === 0
      ? "Enter a display name."
      : undefined;

  function handleChange(event) {
    setDisplayName(event.target.value);
    setShowError(false);
    setStatusMessage("");
  }

  function handleSubmit(event) {
    event.preventDefault();
    setShowError(true);

    if (!normalizedName) {
      setStatusMessage("");
      return;
    }

    setDisplayName(normalizedName);
    setSavedName(normalizedName);
    setShowError(false);
    setStatusMessage("Profile settings saved.");
  }

  function handleCancel() {
    setDisplayName(savedName);
    setShowError(false);
    setStatusMessage("");
  }

  return (
    <section className="profile-settings" aria-label="Profile settings demo">
      <Card
        title="Profile settings"
        description="Update the name shown on your profile."
      >
        <form className="profile-form" onSubmit={handleSubmit} noValidate>
          <TextField
            id="display-name"
            name="displayName"
            label="Display name"
            autoComplete="name"
            value={displayName}
            onChange={handleChange}
            hint="Use a name between 1 and 60 characters."
            error={error}
            maxLength={60}
            required
          />

          <div className="profile-form__actions">
            <Button
              variant="secondary"
              type="button"
              onClick={handleCancel}
              disabled={!hasChanges}
            >
              Cancel
            </Button>

            <Button type="submit" disabled={!hasChanges}>
              Save changes
            </Button>
          </div>
        </form>

        <p className="profile-status" role="status" aria-live="polite">
          {statusMessage}
        </p>
      </Card>
    </section>
  );
}
```

Create `src/features/profile/profile-settings.css`:

```css
.profile-settings {
  max-width: 640px;
}

.profile-form {
  display: grid;
  gap: var(--space-6);
}

.profile-form__actions {
  display: flex;
  justify-content: flex-end;
  gap: var(--space-3);
}

.profile-status {
  min-height: 1.5em;
  margin: var(--space-4) 0 0;
  color: var(--color-muted);
  font-size: 0.9rem;
}

@media (max-width: 480px) {
  .profile-form__actions {
    flex-direction: column-reverse;
  }

  .profile-form__actions .ui-button {
    width: 100%;
  }
}
```

The feature owns profile values and validation. The shared field owns label and description wiring, while the shared buttons own common visuals and native button behavior. The shared components do not import the profile feature.

## 7. Assemble the app

Create `src/app/App.jsx`:

```jsx
import ProfileSettings from "../features/profile/ProfileSettings.jsx";
import "./tokens.css";
import "./app.css";

export default function App() {
  return (
    <div className="app-shell">
      <header className="app-header">
        <p className="app-header__eyebrow">Component system example</p>
        <h1>Account</h1>
        <p>Reusable UI components composed into a feature.</p>
      </header>

      <main>
        <ProfileSettings />
      </main>
    </div>
  );
}
```

Create `src/app/app.css`:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  min-width: 320px;
  min-height: 100vh;
  margin: 0;
}

button,
input {
  font: inherit;
}

.app-shell {
  width: min(100% - 2rem, 900px);
  margin-inline: auto;
  padding-block: 3rem 5rem;
}

.app-header {
  margin-bottom: 2rem;
}

.app-header__eyebrow {
  margin: 0 0 var(--space-2);
  color: var(--color-primary);
  font-size: 0.8rem;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}

.app-header h1 {
  margin: 0 0 var(--space-2);
  font-size: clamp(2rem, 5vw, 3rem);
}

.app-header > p:last-child {
  margin: 0;
  color: var(--color-muted);
  line-height: 1.55;
}
```

Create `src/main.jsx`:

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./app/App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

The existing app entry page must provide an element with `id="root"`, as in a standard Vite React starter.

## 8. Try the public component APIs

1. The profile form starts with a saved display name, so **Save changes** and **Cancel** are disabled.
2. Edit the text. Both buttons become available because the form now has unsaved changes.
3. Select **Cancel**. The feature restores the saved value.
4. Enter a different non-empty name and select **Save changes**. The feature stores the value and announces a confirmation.
5. Clear the field and submit. The field displays an error and marks itself invalid.
6. Use the form with a keyboard. The input has a visible label, and focus is visible on interactive controls.

The reusable components expose a small contract:

- `Button`: native button props, `type`, `disabled`, `variant`, `size`, and `children`.
- `TextField`: a unique `id`, visible `label`, controlled `value` and `onChange`, optional hint and error, plus native input props.
- `Card`: a title, optional description, and composable children.

## 9. What would change before publishing a package?

This example is private app code. If other projects need it as a package, add a deliberate public entry point, documented exports, supported React compatibility, a build output, styling instructions, and a release process. Configure the build so React is provided by the consuming application. The exact package fields and build configuration depend on the chosen toolchain.
