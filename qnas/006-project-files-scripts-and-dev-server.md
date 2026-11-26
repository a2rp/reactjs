# 006. Q&A: Project files, scripts, and the development server

Use these questions to understand the files and commands in a typical Vite React starter. A generated template may change over time, so inspect your own project's files when names or scripts differ.

## Project structure

### 1. What is the role of the project-root `index.html` in Vite?

It is the HTML document Vite serves as the application entry page. It usually contains the root element for React and loads the JavaScript entry module. In a Vite project, this file is normally at the project root, not hidden in `public/`.

### 2. What is the purpose of `src/main.jsx`?

It is commonly the JavaScript entry module. It imports React, React DOM, the app component, and CSS, then creates a React root and renders the component tree.

### 3. How are `main.jsx` and `App.jsx` different?

`main.jsx` connects the app to the document and starts rendering. `App.jsx` commonly exports the top-level app component. As the project grows, that component can compose other components.

### 4. What are `index.css` and `App.css` usually for?

A starter often uses `index.css` for global styles and `App.css` for styles related to the app component. These are conventions, not special React requirements. Check the imports to see which styles are actually loaded.

### 5. What goes in `vite.config.js`?

Project-specific Vite configuration, such as plugins or custom build and server options. A small starter may not need a custom config file. Do not change it just because a tutorial has one.

### 6. What is `package.json`?

It records project metadata, dependencies, and named scripts. npm reads it when you install packages or run commands such as `npm run dev`.

### 7. What does `package-lock.json` record?

It records npm's resolved dependency tree so later installs can reproduce the chosen package versions more consistently. Commit this file in an npm-managed app.

### 8. Why should `node_modules/` not be committed?

It contains installed package files and can be recreated from the package manifest and lockfile. It is large generated dependency data, not the application's source code.

### 9. What is `dist/`?

It is the common output directory for a Vite production build. It contains generated assets. Edit source files instead; the next build recreates this output.

## The browser startup path

### 10. What is the simple loading path from URL to React UI?

The browser requests the root page, Vite serves `index.html`, the page loads the entry module such as `src/main.jsx`, and that module creates a React DOM root and renders the app component.

### 11. Why does React need a root element in the HTML?

React DOM needs a DOM node where it can mount the React-managed interface. A starter often provides an element with `id="root"` and passes that element to `createRoot`.

### 12. Is JSX run directly by the browser?

No. JSX is transformed by the configured project tooling. The browser loads the output JavaScript modules, not the raw JSX syntax as written in source.

## npm scripts and the dev server

### 13. What is an npm script?

It is a named command in `package.json`'s `scripts` section. For example, a Vite project commonly defines `dev`, `build`, and `preview`.

### 14. How can I see the scripts available in this project?

Open `package.json` and inspect its `scripts`, or run `npm run` from the project directory. Script names are project-specific, so do not assume every repository uses the same names.

### 15. Why must I run `npm run dev` from the project directory?

npm needs to find the `package.json` that defines the script and the dependencies installed for that project. If npm reports a missing script, first check the current directory.

### 16. What does the Vite `dev` script do?

It starts the development server. The terminal prints the address to open in a browser and reports startup or compilation errors.

### 17. What is hot module replacement?

Hot module replacement (HMR) updates supported modules in the running page when source files change, often without a full page reload. Some changes trigger a reload, and component state may reset. HMR is a development convenience, not proof that the production build works.

### 18. How do I stop the dev server?

Focus the terminal where the server is running and press Ctrl+C. Closing the browser tab alone does not stop the terminal process.

### 19. What is the default Vite development port?

The common default is 5173, but the actual address printed in the terminal is authoritative. If that port is occupied, Vite can choose another available port unless configured to exit instead.

### 20. Why does Vite listen on localhost by default?

Localhost serves development on the local machine. Vite can be configured to listen on other interfaces, but that can make the server reachable from a network. Only expose it when needed and when you understand the access scope.

## Assets, environment, and production

### 21. When should I put a file in `public/`?

Use the public directory for files that should be served or copied unchanged, such as a file that must keep a fixed name. A file at `public/icon.png` is normally referenced as `/icon.png`. For typical source assets, importing them from source lets the build process handle their URLs.

### 22. What happens to imported assets in a production build?

The build tool can process imported assets and adjust their output URLs, including adding hashed filenames. Public-directory assets are copied as-is and keep their names.

### 23. What does `npm run build` do?

In a typical Vite starter, it builds production assets into the output directory, commonly `dist/`. It does not publish the app or provide a hosting destination.

### 24. What does `npm run preview` do?

It serves the built output locally so you can inspect it. It is not a production deployment server. Use the printed preview URL and stop the process with Ctrl+C when finished.

### 25. Are `VITE_*` environment variables secret?

No. Vite exposes variables with the `VITE_` prefix to browser-side code, so those values can be inspected by users. Do not put passwords, private API keys, or other credentials in them. Keep secrets in a trusted server-side environment.

### 26. Why might I need to restart Vite after changing an environment file?

Vite loads environment files when the process starts. Restart the dev server so it reads the updated values.

## Troubleshooting

### 27. npm says that `dev` is not a script. What should I check?

Confirm that you are in the project directory, open that project's `package.json`, and use a script name that is actually listed there.

### 28. The browser cannot open the local page. What should I check first?

Check whether the server is still running and use the exact URL printed in the terminal. If it chose a different port, an older URL may no longer point to this process.

### 29. The page is blank but the terminal says the server is running. Where should I look?

Check the browser Console for JavaScript runtime or module errors. Then inspect the Network panel for missing scripts, styles, or assets. The terminal and browser report different layers of problems.

### 30. Why should I not fix a problem by editing `node_modules/` or `dist/`?

Those are generated directories. Package installation can replace `node_modules/`, and a build recreates `dist/`. Find and change the responsible source file or configuration, then rerun the appropriate command.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="005-choose-a-react-project-setup.md">005 - Q&amp;A: Choose a React project setup</a></td><td><a href="../lessons/006-project-files-scripts-and-dev-server.md">Lesson</a></td><td><a href="../examples/006-project-files-scripts-and-dev-server.md">Example</a></td><td><a href="007-jsx-syntax-and-markup.md">007 - Q&amp;A: JSX syntax and markup</a></td></tr>
  </tbody>
</table>
