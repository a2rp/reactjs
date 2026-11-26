# 009. JSX styles, event props, and DOM properties - Example

## Goal

Build a small course card that uses a CSS class for its main design, one dynamic inline style, native DOM props, and event-handler functions. The example logs button actions to the browser console; it does not add state or save data.

## Files

Use the source files from your React project:

- src/App.jsx
- src/App.css

This assumes the project setup from the earlier chapters already loads the root App component and supports importing CSS from JavaScript.

## src/App.jsx

~~~jsx
import './App.css';

const course = {
  id: 'react-essentials',
  title: 'React Essentials',
  level: 'Beginner',
  accent: '#2563eb',
  contact: 'learn@example.test',
  available: true,
};

export default function App() {
  function handlePreview() {
    console.log('Preview requested: ' + course.title);
  }

  function handleSelect(courseId) {
    console.log('Selected course: ' + courseId);
  }

  return (
    <main className="page">
      <article
        className="course-card"
        data-course-id={course.id}
        data-level={course.level}
        style={{ borderTopColor: course.accent }}
      >
        <p className="course-card__eyebrow">COURSE</p>
        <h1 className="course-card__title">{course.title}</h1>
        <p className="course-card__level">Level: {course.level}</p>

        <p id="course-availability" className="course-card__availability">
          {course.available ? 'Enrollment is open.' : 'Enrollment is closed.'}
        </p>

        <label className="course-card__label" htmlFor="course-contact">
          Course contact
        </label>
        <input
          className="course-card__input"
          id="course-contact"
          name="courseContact"
          type="email"
          value={course.contact}
          readOnly
        />

        <div className="course-card__actions">
          <button
            className="course-card__button"
            type="button"
            aria-describedby="course-availability"
            disabled={!course.available}
            onClick={handlePreview}
          >
            Preview course
          </button>

          <button
            className="course-card__button course-card__button--secondary"
            type="button"
            disabled={!course.available}
            onClick={() => handleSelect(course.id)}
          >
            Select course
          </button>
        </div>
      </article>
    </main>
  );
}
~~~

## src/App.css

~~~css
:root {
  font-family: system-ui, sans-serif;
  color: #172033;
  background: #f1f5f9;
}

* {
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

.page {
  display: grid;
  min-height: 100vh;
  place-items: center;
  padding: 2rem;
}

.course-card {
  width: min(100%, 34rem);
  border: 1px solid #cbd5e1;
  border-top: 0.4rem solid;
  border-radius: 1rem;
  padding: 2rem;
  background: #ffffff;
  box-shadow: 0 1rem 2rem rgb(15 23 42 / 10%);
}

.course-card__eyebrow {
  margin: 0 0 0.5rem;
  color: #475569;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.12em;
}

.course-card__title {
  margin: 0;
  font-size: clamp(1.75rem, 5vw, 2.5rem);
}

.course-card__level {
  margin: 0.75rem 0;
  color: #475569;
}

.course-card__availability {
  margin: 1.25rem 0;
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
  background: #eff6ff;
}

.course-card__label {
  display: block;
  margin-bottom: 0.4rem;
  font-weight: 600;
}

.course-card__input {
  width: 100%;
  margin-bottom: 1.25rem;
  border: 1px solid #94a3b8;
  border-radius: 0.4rem;
  padding: 0.65rem 0.75rem;
  background: #f8fafc;
}

.course-card__actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
}

.course-card__button {
  border: 1px solid #1d4ed8;
  border-radius: 0.5rem;
  padding: 0.7rem 1rem;
  color: #ffffff;
  background: #1d4ed8;
  cursor: pointer;
}

.course-card__button--secondary {
  color: #1d4ed8;
  background: #ffffff;
}

.course-card__button:hover:not(:disabled) {
  filter: brightness(0.92);
}

.course-card__button:focus-visible {
  outline: 3px solid #f59e0b;
  outline-offset: 3px;
}

.course-card__button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}
~~~

## What to notice

- The article and its children use className to connect markup to reusable CSS rules.
- borderTopColor is a dynamic style value from the course data. The rest of the visual design stays in the stylesheet.
- data-course-id and data-level remain hyphenated because they are data attributes.
- htmlFor on the label matches the input id, so the label names that control.
- readOnly is a boolean DOM prop. It makes the displayed contact value non-editable.
- disabled receives a boolean expression. Changing available to false disables both actions.
- aria-describedby connects the preview button to the availability message.
- onClick={handlePreview} passes a function. The arrow function on the second button delays handleSelect(course.id) until activation.
- Both actions use native button elements, so the browser supplies button semantics and keyboard activation.

## Try these changes

1. Change course.accent and observe the card's top border update without changing its CSS class.
2. Change course.available to false and observe that both buttons become disabled.
3. Replace onClick={handlePreview} with onClick={handlePreview()}. Notice that the function runs while rendering and its return value is not a handler. Restore the function reference.
4. Temporarily change htmlFor to a different id and observe that clicking the label no longer focuses the input. Restore the matching value.
5. Inspect the card in browser developer tools and find its data-course-id and data-level attributes.
6. Click each button and inspect the console messages.

## Expected result

The page shows a centered course card with a blue top border, course information, a labeled read-only email field, and two buttons. Clicking either available button writes a message to the browser console. The browser console should not show a React warning about these props.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="008-jsx-expressions-attributes-and-escaping.md">008 - Example: Render data and expressions in JSX</a></td><td><a href="../lessons/009-jsx-styles-events-and-dom-properties.md">Lesson</a></td><td><a href="../qnas/009-jsx-styles-events-and-dom-properties.md">Q&amp;A</a></td><td><a href="010-fragments-and-multiple-elements.md">010 - Fragments and returning multiple elements - Example</a></td></tr>
  </tbody>
</table>
