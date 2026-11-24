# 006. Example: Trace and run a Vite React project

## Goal

Use the Vite React project created in lesson 005 to trace its important files, make a small source edit, observe the development server update the page, and inspect a production build locally.

The exact starter content can vary by Vite template version. Keep your generated entry files unless you have a reason to change them.

## Step 1: Open the project and inspect its files

Open the terminal in the project directory created in the previous lesson. It should be the folder that contains `package.json`.

A common starter has a structure similar to:

```text
react-learning-lab/
├── index.html
├── package.json
├── package-lock.json
├── public/
├── src/
│   ├── App.jsx
│   ├── App.css
│   ├── index.css
│   └── main.jsx
├── vite.config.js
└── node_modules/
```

You may see extra assets or configuration files. Open these files and identify their roles:

1. In the root `index.html`, find the element reserved for the React UI and the module script that loads `src/main.jsx`.
2. In `src/main.jsx`, find the import of `App.jsx`, the call that creates the React root, and the render call.
3. In `src/App.jsx`, find the component that the starter renders.
4. In `package.json`, inspect the project scripts.
5. Notice that `node_modules/` contains installed dependencies. Do not edit it.

The startup path is:

```text
index.html -> src/main.jsx -> React DOM root -> App component -> browser UI
```

## Step 2: Replace only the starter component

Open `src/App.jsx` and replace its component with this small page:

```jsx
export default function App() {
  return (
    <main>
      <h1>React project is running</h1>
      <p>
        The HTML document loaded the entry module, and React rendered this
        component into the page.
      </p>
    </main>
  );
}
```

If the starter's `App.jsx` imports an app-specific CSS file that you no longer need, remove that unused import from this file. Keep the generated `src/main.jsx` and its global CSS import unchanged for now.

## Step 3: Start the development server

From the directory containing `package.json`, run:

```sh
npm run dev
```

Open the exact local URL shown in the terminal. You should see:

> React project is running

and the explanatory paragraph below it.

Keep the terminal process running while using the page. To stop the server later, focus its terminal and press Ctrl+C. Closing the browser tab does not stop the server.

## Step 4: Observe a source update

Leave the server running and change the heading in `App.jsx` to:

```jsx
<h1>My first React development loop</h1>
```

Save the file. The browser should update through Vite's development reload or hot module replacement. This confirms the current source is connected to the running project. It does not prove that every route or the production build is correct.

Restore the original heading if you want the page to match the expected result above.

## Step 5: Inspect the available scripts

Open `package.json` and find the `scripts` section. A Vite starter commonly defines commands similar to:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

Your actual file is authoritative; a newer template may add or rename scripts. You can ask npm to list the available scripts:

```sh
npm run
```

Run scripts from the project folder, where npm can find the right `package.json` and installed tools.

## Step 6: Build and preview the output

Stop the dev server with Ctrl+C. From the project directory, run:

```sh
npm run build
```

The command should finish and create a production output folder, commonly `dist/`. Inspect it if you want to see the generated files, but do not edit them; a later build recreates the folder.

Preview the built output locally:

```sh
npm run preview
```

Open the address printed in the terminal and confirm the heading and paragraph appear. Stop preview with Ctrl+C.

Preview only helps inspect the production build on your machine. It does not publish the project or set up hosting.

## What belongs where?

| Change you want to make | File or folder to inspect |
| --- | --- |
| Change the page content | `src/App.jsx` |
| Change global base styles | `src/index.css` |
| Change app-specific styles | The app stylesheet imported by `App.jsx`, often `src/App.css` |
| Change document title or metadata | Root `index.html` |
| Add a source image that should be processed by the build | An asset imported from `src/` |
| Add a fixed-name file served as-is | `public/`, referenced from the site root |
| Change npm scripts or dependencies | `package.json` |
| Change Vite-specific behavior | `vite.config.js`, if the project needs it |

Do not put passwords or private credentials in variables with the `VITE_` prefix. Those values can be included in browser code.

## Expected result

You have identified the handoff from root HTML to the JavaScript entry and then to the React component. The development server displays your edit, and the production build can be served locally through the preview command.

The dev server and preview are separate terminal processes. Use the URL printed by the active process rather than assuming both use the same port.

## Troubleshooting

- **npm says a script is missing:** Check that the terminal is in the project folder and inspect the scripts in `package.json`.
- **The browser does not connect:** Confirm the server is still running and open its printed URL. The port can change if the usual one is occupied.
- **The page shows a module error:** Check the browser Console and the terminal. A misspelled import or missing file can prevent the component from loading.
- **The edit does not appear:** Save the source file and confirm that the running page belongs to this project. Check for errors in the terminal.
- **The build fails:** Read the first error reported by the build command and check source imports and syntax. Do not try to fix it by editing `dist/`.
- **A public file returns a 404:** Check that it is inside the root `public/` folder and reference it from the site root, such as `/icon.png`.

## Completion checklist

- [ ] I can point to the project's root `index.html`.
- [ ] I can explain how `main.jsx` loads and renders `App.jsx`.
- [ ] I changed the app source and saw the browser update.
- [ ] I found the project's actual npm scripts.
- [ ] I built and previewed the production output locally.
- [ ] I know not to edit `node_modules/` or generated `dist/`.
- [ ] I know that `VITE_*` values are client-visible, not private secrets.

## Chapter navigation

| Previous chapter | Lesson | Q&A | Next chapter |
| --- | --- | --- | --- |
| [005 - Choose a React project setup](../lessons/005-choose-a-react-project-setup.md) | [Open lesson](../lessons/006-project-files-scripts-and-dev-server.md) | [Open Q&A](../qnas/006-project-files-scripts-and-dev-server.md) | [007 - JSX syntax and markup](../lessons/007-jsx-syntax-and-markup.md) |
