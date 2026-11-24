# 007. Example: Convert HTML-like markup into valid JSX

## Goal

Convert a small course overview from HTML-like markup into valid JSX, place it in a React component, and identify why each correction is needed.

## Starting point: markup copied from HTML

Imagine this markup was copied into a JavaScript component:

```html
<h1 class="course-title">React foundations</h1>
<img src="/course-cover.png" alt="A notebook labeled React">
<p>Learn the building blocks of a React interface.
<ul>
  <li>Describe content with markup
  <li>Close and nest elements
</ul>
<footer>Practice one small change each day.</footer>
```

This fragment is valid HTML, but it is not valid as one JSX expression. It has adjacent top-level elements, uses HTML's `class` attribute, and relies on HTML's void-element and optional-tag parsing where JSX expects explicit syntax.

## Step 1: Choose the root structure

The page needs a main region and a footer as siblings. A Fragment can group both without adding a meaningless wrapper element:

```jsx
<>
  <main>
    {/* Main page content */}
  </main>
  <footer>
    {/* Closing page content */}
  </footer>
</>
```

A Fragment is one way to make the JSX expression have a single root. A meaningful parent such as a page-level container would also work if the document structure called for one.

## Step 2: Fix the attributes and close every tag

In JSX:

- Write `className` instead of `class`.
- Close an element with children using its matching closing tag.
- Self-close a void element such as `img` with `/>`.
- Nest tags in the same order they open.

## Step 3: Write the complete component

Replace the content of `src/App.jsx` with:

```jsx
export default function App() {
  return (
    <>
      <main>
        <article className="course-card">
          <header>
            <h1 className="course-title">React foundations</h1>
          </header>

          <p>
            Learn the building blocks of a React interface.
          </p>

          <img
            src="/course-cover.png"
            alt="A notebook labeled React"
          />

          <section aria-labelledby="topics-heading">
            <h2 id="topics-heading">What you will practice</h2>
            <ul>
              <li>Describe content with markup</li>
              <li>Close and nest elements</li>
              <li>Build one small interface at a time</li>
            </ul>
          </section>
        </article>
      </main>

      <footer>
        <p>Practice one small change each day.</p>
      </footer>
    </>
  );
}
```

This component uses a Fragment as its single JSX root, then meaningful HTML elements for the content. Every open tag has a matching close tag, except the void `img`, which is self-closing. The `className` properties and the `aria-labelledby` attribute use JSX-compatible names.

The image path is an example. Add a matching file at `public/course-cover.png` for it to appear; if the file is absent, the browser will show a missing-image result. The rest of the component works without that image.

## Step 4: Render it in the starter

The Vite React starter should already load `src/App.jsx` from `src/main.jsx`. Leave that entry point in place. Start the dev server from the folder that contains `package.json`:

```sh
npm run dev
```

Open the printed local address. You should see the heading, summary, topic list, and footer. If the image is not present, use the browser Console or Network panel to see the missing request; add the sample file or temporarily remove the `img` element.

## Why the final JSX is valid

| Correction | Reason |
| --- | --- |
| The main content and footer are inside `<></>` | One JSX expression needs a single root; a Fragment adds no wrapper element to the DOM. |
| `class` became `className` | JSX uses the DOM property name for the class attribute. |
| The image ends with `/>` | JSX requires explicit closure for void elements. |
| Every `li`, `ul`, `section`, and other paired element closes | JSX requires closed and properly nested tags. |
| The elements use meaningful structure | Semantic markup makes the document easier to understand and navigate. |

## Try these changes

1. Replace the Fragment with a meaningful `<div>` wrapper, then inspect the Elements panel to see the extra DOM node.
2. Remove one `</li>` and read the first error reported by the development tools. Restore it afterward.
3. Change `className` back to `class` and inspect the warning or error from the toolchain.
4. Add another topic as a properly closed `<li>`.
5. If you add a custom component later, give its function and JSX tag an uppercase name.

## Common mistakes

- **Using two top-level sibling tags without a wrapper:** use an element, Fragment, or an explicit array when appropriate.
- **Writing `class` in JSX:** use `className`.
- **Leaving off the slash on an image:** write `<img ... />`.
- **Closing a parent before its child:** close the inner element first.
- **Adding a generic wrapper automatically:** prefer a meaningful element or a Fragment when an extra node is unnecessary.
- **Assuming the example image already exists:** create the file in `public/` or remove the image markup.
- **Changing `main.jsx` to fix a markup error:** first check the component's opening and closing tags in `App.jsx`.

## Completion checklist

- [ ] The component returns a single JSX expression.
- [ ] Paired tags are explicitly closed and correctly nested.
- [ ] Void tags are self-closing.
- [ ] The class property uses `className`.
- [ ] The semantic content appears in the browser.
- [ ] I can explain what a Fragment adds and what it leaves out.
- [ ] I know whether the sample image file exists in my project.

## Chapter navigation

| Previous chapter | Lesson | Q&A | Next chapter |
| --- | --- | --- | --- |
| [006 - Project files, scripts, and the development server](../lessons/006-project-files-scripts-and-dev-server.md) | [Open lesson](../lessons/007-jsx-syntax-and-markup.md) | [Open Q&A](../qnas/007-jsx-syntax-and-markup.md) | [008 - JavaScript expressions, attributes, and escaping in JSX](../lessons/008-jsx-expressions-attributes-and-escaping.md) |
