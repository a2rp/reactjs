# 005. Example: Choose and start a React learning project

## Goal

Work through a setup decision for a beginner who wants to learn React in a local browser app. By the end, you will have created and opened a small React starter project and previewed its production build.

This example chooses Vite because the goal is to learn React fundamentals with a simple client-rendered project. For a new production app, evaluate a React framework first.

## The scenario

Maya knows basic HTML, CSS, and JavaScript. She wants to follow the early React lessons on her laptop. Her project needs:

- A browser interface.
- A local development server.
- JSX support.
- A simple project structure that she can inspect in the next lesson.

She does not currently need native mobile UI, a backend, server rendering, or production routing. A Vite React starter is a reasonable fit for this learning goal.

## Step 1: Check the local tools

Open a terminal and check that Node.js and npm are available:

```sh
node --version
npm --version
```

The commands should print version numbers. The Vite version installed by the project creator may require a recent Node.js release. If the setup reports that the installed Node version is unsupported, install a supported Node release and open a fresh terminal before continuing.

## Step 2: Create a JavaScript React project

Run:

```sh
npm create vite@latest react-learning-lab -- --template react
```

This asks npm to run the Vite project creator, creates the `react-learning-lab` project directory, and selects the JavaScript React template. If the creator asks follow-up questions, select React and JavaScript. The prompt wording may differ between releases.

Move into the new folder, install its dependencies, and start the development server:

```sh
cd react-learning-lab
npm install
npm run dev
```

Keep the terminal open. The last command prints a local address. Open that address in a browser.

## Step 3: Confirm the starter runs

The starter page should load in the browser. You have now confirmed that:

- The development server started.
- The browser received the app.
- The React starter rendered an interface.

If the page does not load, read the terminal output first. Check that the address was copied correctly, that the dev server is still running, and that Node.js was supported. If the page itself loads with an error, inspect the browser Console.

Stop the development server with Ctrl+C in its terminal when you are done.

## Step 4: Preview a production build

Stop the development server with Ctrl+C and make sure the terminal is in the project directory before running:

```sh
npm run build
npm run preview
```

The build command creates production assets. Preview serves those assets locally; the terminal prints the address to open. Confirm the page appears. Stop preview with Ctrl+C when finished.

This checks the built output on your computer. It does not publish the site or choose a hosting provider.

## What this project includes

For this scenario, the project has React, React DOM, Vite, JSX transformation, a development server, and a production build command. Keep the generated package lockfile in version control so later installs use the dependency versions recorded for the project.

The plain starter is client-rendered. It does not automatically include application routes, server-side data loading, authentication, a database, or deployment configuration. Add features deliberately when the app needs them.

## Compare the decision with other scenarios

| If the goal changes to... | Reconsider this choice |
| --- | --- |
| A new multi-route web app with integrated data loading or server rendering | Evaluate a React framework before building on the starter. |
| One interactive widget inside an existing website | Add React to the existing site or mount a root just for that area. |
| Native Android or iOS screens | Start with React Native tooling such as Expo. |
| A quick experiment with one component | Use an online React sandbox instead of installing a project. |
| Static checking from the beginning | Choose the Vite React TypeScript template rather than the JavaScript template. |

For a TypeScript learning project, create the starter with:

```sh
npm create vite@latest react-learning-lab -- --template react-ts
```

Do not run both starter commands in the same directory. Pick one template when creating a project.

## Troubleshooting

### The terminal says the Node.js version is unsupported

Use a Node.js release supported by the current Vite version, then retry the project creation. Setup tools change their minimum versions over time.

### The project folder already exists

Choose a different new directory name or inspect the existing folder before using it. Do not overwrite files you have not reviewed.

### The browser cannot reach the local address

Make sure `npm run dev` is still running and use the exact address printed by the terminal. The development address is local to your machine.

### The command `npm run build` is missing

Make sure the terminal is inside the project folder created by Vite. The scripts belong to that project's package configuration, not to a parent folder.

### I need routing or server data now

Revisit the original decision. A plain Vite starter does not automatically supply those application features. Evaluate a React framework or intentionally select and configure the tools the app requires.

## Completion checklist

- [ ] Node.js and npm print version numbers.
- [ ] The Vite project creator makes a new React project folder.
- [ ] Dependencies install successfully.
- [ ] The development server prints a local address.
- [ ] The starter page opens in a browser.
- [ ] The production build command completes.
- [ ] The previewed production build opens locally.
- [ ] You can explain why this learning scenario uses Vite and what the starter does not include.
- [ ] You can name a different setup to evaluate if the project becomes a full production web app, an existing-site integration, or a native mobile app.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="004-javascript-and-browser-prerequisites.md">004 - Example: Use JavaScript data and functions in a React component</a></td><td><a href="../lessons/005-choose-a-react-project-setup.md">Lesson</a></td><td><a href="../qnas/005-choose-a-react-project-setup.md">Q&amp;A</a></td><td><a href="006-project-files-scripts-and-dev-server.md">006 - Example: Trace and run a Vite React project</a></td></tr>
  </tbody>
</table>
