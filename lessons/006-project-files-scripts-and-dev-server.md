# 006. Project files, scripts, and the development server

## Learning goals

By the end of this lesson, you should be able to:

- Recognize the important files in a basic Vite React project.
- Trace how the browser document loads the React application.
- Read the common npm scripts and run them from the correct folder.
- Start and stop the development server and understand its printed address.
- Explain fast refresh, a production build, and a local build preview.
- Keep generated dependencies and private values out of the wrong places.

This lesson describes a typical JavaScript Vite React starter. Templates and tool versions evolve, so your generated project may have extra files or slightly different starter content.

## A typical project shape

A starter project may contain files like these:

```text
react-learning-lab/
├── index.html
├── package.json
├── package-lock.json
├── vite.config.js
├── public/
└── src/
    ├── main.jsx
    ├── App.jsx
    ├── index.css
    └── App.css
```

The exact tree can vary. Understand each file's responsibility instead of memorizing every template detail.

| File or folder | Purpose |
| --- | --- |
| `index.html` | The HTML document Vite serves as the browser entry page. It contains the root element and loads the JavaScript module entry. |
| `src/main.jsx` | The JavaScript entry module. It imports React, React DOM, app code, and usually global CSS, then creates the React root. |
| `src/App.jsx` | A starting React component. As the app grows, it can render or compose other components. |
| `src/index.css` | Often contains global styles such as body defaults and shared typography. |
| `src/App.css` | Often contains styles used by the starter app component. |
| `public/` | Static files served or copied unchanged, typically addressed from the site root. |
| `src/assets/` | A common location for source assets that the app imports and the build tool can process. |
| `package.json` | Project metadata, dependency declarations, and named scripts. |
| `package-lock.json` | The npm lockfile recording the resolved dependency tree for repeatable installs. Commit it for an npm project. |
| `node_modules/` | Installed packages. npm creates this locally; it is not source code and should not be committed. |
| `vite.config.js` | Optional project-specific Vite configuration. A small starter may not need custom configuration. |
| `dist/` | Common output directory created by a production build. It is generated output, not the place to edit source files. |

Vite treats the project root's `index.html` as a first-class entry point. In a Vite project it is not normally hidden inside the `public` directory.

## Follow the browser startup path

A simplified loading path looks like this:

```text
browser requests /
      |
      v
Vite serves project-root index.html
      |
      v
index.html loads src/main.jsx
      |
      v
main.jsx creates a React DOM root
      |
      v
React renders the App component into the root element
```

A starter's `index.html` commonly includes an element such as:

```html
<div id="root"></div>
```

The React entry module finds that element and mounts the component tree. A simplified example is:

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

This shows the roles of the files, not a requirement to replace your generated entry file. Keep the template's entry code intact until you understand what it does. The later chapter on creating a root explains this React API in detail.

## Understand package scripts

The `scripts` section of `package.json` gives short names to project commands. A Vite starter commonly includes:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

- `npm run dev` starts the development server.
- `npm run build` builds production assets.
- `npm run preview` serves the built assets locally for inspection.

Run scripts from the project folder containing `package.json`. npm looks up the named script in that file and makes the project's installed command-line tools available while running it.

To see the scripts in a project, open `package.json` or run:

```sh
npm run
```

Do not assume every project has the exact same script names. The project file is the source of truth.

## Start and stop the development server

From the project folder, run:

```sh
npm run dev
```

Vite prints a local URL in the terminal, often using port 5173. Open the exact URL it prints; if that port is already occupied, Vite can try another port. Do not assume a particular port without checking the terminal output.

The development server serves files and watches for source changes. When a supported file changes, Vite can update the browser quickly through hot module replacement (HMR) or a full reload. This feedback loop is for development; it is not the deployed production server.

Keep the terminal process running while using the app. Press Ctrl+C in that terminal to stop it. Closing the browser tab does not stop the server.

By default, Vite listens on localhost for local development. The `--host` option can make the server listen on other network interfaces. Do not expose a development server to a network unless you specifically need that and understand who can reach it.

## Edit source files, not generated output

Make application changes under `src/` and edit the root `index.html` for document-level metadata or markup. Do not edit files inside `node_modules/`; package installation can replace them. Do not edit the generated `dist/` directory; the next build recreates it from source.

For assets:

- Prefer importing an image or font from source code when the app uses it as a module asset.
- Use `public/` for files that need a fixed name or are served as-is, such as a root-level `robots.txt`.
- A file at `public/icon.png` is normally referenced as `/icon.png`, not `/public/icon.png`.

The build tool may hash imported asset filenames in production. A public file keeps its name and is copied as-is.

## Use environment variables carefully

Vite exposes variables with the `VITE_` prefix to browser-side code. For example, `VITE_SITE_TITLE` can be read by client code, but it is not secret. Any value included in a browser bundle can be inspected by a visitor.

Never place database passwords, private API keys, signing secrets, or other credentials in a `VITE_*` variable. Those belong in a trusted server-side environment. Local-only `.env.*.local` files should stay out of version control according to the project's ignore rules. Restart the dev server after changing environment files so they are loaded again.

## Build and preview

When you want to inspect the production output:

```sh
npm run build
npm run preview
```

The build step reads the source and writes generated assets, typically into `dist/`. Preview serves those built files locally, often on a different port than the development server. Open the address printed by preview and check that the page and its assets load.

Preview is a local inspection tool. It does not deploy the project, configure a domain, or create a backend. A framework or hosting provider may use a different build and deployment workflow.

## Troubleshoot by layer

| Symptom | First place to check |
| --- | --- |
| `npm run dev` says no script exists | Confirm the terminal is in the folder with the expected `package.json`, then inspect its `scripts`. |
| npm says a package or command is missing | Confirm dependencies were installed in this project with `npm install`. |
| The expected local port does not open | Read the terminal output; Vite may have selected a different port or failed to start. |
| The browser is blank or shows a runtime error | Check the browser Console and inspect failed imports or rendering errors. |
| A source edit does not appear | Confirm you edited a file used by the running app and that the server is still running. |
| A CSS or image file is missing after build | Check its import or public path and inspect browser Network requests. |
| A value seems missing in client code | Check its name and whether it has the required `VITE_` prefix; never add a secret to fix it. |

The terminal reports build-tool and compilation problems. The browser Console reports JavaScript runtime problems. The Network panel helps show whether documents, scripts, styles, and assets loaded.

## Common mistakes

- **Running npm from a parent directory:** scripts are defined by the project's own `package.json`.
- **Editing `node_modules` or `dist`:** change source files and rebuild instead.
- **Assuming the local port:** use the URL printed by the server.
- **Closing the terminal and expecting the server to continue:** stop and start the process from the terminal as needed.
- **Treating HMR as proof of production correctness:** inspect a production build with the build and preview scripts too.
- **Putting credentials in frontend variables:** `VITE_*` values are part of client-side code.
- **Putting ordinary app assets in `public/` by habit:** import assets from source unless they need the guarantees of the public directory.

## Key takeaways

- Root `index.html` leads to the app's JavaScript entry, which creates a React root and renders the component tree.
- `package.json` names project scripts and dependencies; the lockfile records resolved npm dependencies.
- The dev server is a local authoring tool. Use its printed URL and stop it with Ctrl+C.
- Edit source, not installed packages or generated output.
- Vite's `public/` files are copied as-is; imported assets can be processed by the build.
- Anything exposed with a `VITE_` prefix is public client data, not a secret.
- Build and preview help inspect production assets locally, but do not deploy them.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [005 - Choose a React project setup](005-choose-a-react-project-setup.md) | [Open Q&A](../qnas/006-project-files-scripts-and-dev-server.md) | [Open Example](../examples/006-project-files-scripts-and-dev-server.md) | [007 - JSX syntax and markup](007-jsx-syntax-and-markup.md) |
