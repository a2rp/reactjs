# 005. Q&A: Choose a React project setup

Use these questions to decide which setup fits your goal and to review the setup choices from lesson 005.

## Choosing a project path

### 1. What should I decide before installing React?

First decide what you are building: trying a small component, learning React locally, starting a full web app, adding an interactive area to an existing site, or building a native mobile app. These goals can use different tools.

### 2. What is React responsible for?

React is the UI library. It lets you describe interfaces with components and update them when their inputs change. React alone does not decide your app's routing, backend, deployment, or project build process.

### 3. What does a build tool do?

A build tool such as Vite serves the project during development, transforms source files such as JSX, and creates production assets. A plain build-tool starter does not automatically provide all application features such as routing and server data loading.

### 4. What does a React framework add?

A framework provides application-level conventions and integrated solutions for features such as routing, data loading, rendering strategies, and deployment. It makes more architectural decisions for you than a minimal build-tool starter.

### 5. Does a framework always require a server?

No. Frameworks can support client-rendered and statically generated applications as well as server-rendered routes. The framework choice and the rendering strategy are related, but they are not the same decision.

### 6. What setup does React recommend for a new full React app?

React recommends starting a new app or website built fully with React using a framework. Starting from scratch with a build tool can still fit learning, embedding React into an existing site, or requirements that a framework does not serve well.

### 7. Why does this course use Vite for its early lessons?

A Vite React starter gives the lessons a small local browser project with JSX support and a development server. Its visible files and scripts make the early React concepts easier to inspect. It is a teaching choice, not a recommendation that every production app should use Vite alone.

### 8. When should I consider a framework instead of a plain Vite starter?

Consider a framework when a new app needs features such as several routes, data loading, SEO support, server rendering, or an integrated deployment approach. Compare the framework's conventions with the app's hosting and team needs.

### 9. Can I use React in a website that already exists?

Yes. You can add React to the existing project or mount a React root in one interactive area. This gradual approach lets you keep unrelated pages and systems in place.

### 10. Is a browser React app the same as a native phone app?

No. A browser app runs as a website. For native Android or iOS interfaces, use React Native tooling such as Expo rather than expecting a browser build tool to produce native screens.

## Language and prerequisites

### 11. Should I start with JavaScript or TypeScript?

Either works. JavaScript keeps the initial learning surface smaller; TypeScript can add static checks for data, props, and functions and may fit a team that already uses it. Use the matching project template and be consistent.

### 12. What do I need before running the Vite setup command?

Install a supported Node.js version, which includes npm. The current Vite release can require a minimum Node.js version. Check with `node --version` and `npm --version`; if the setup tool reports an unsupported version, update Node before continuing.

### 13. What does this command create?

```sh
npm create vite@latest react-learning-lab -- --template react
```

It runs the current Vite project creator, names the new project folder `react-learning-lab`, and selects the JavaScript React template. Tool prompts and options can change over time.

### 14. How do I request the TypeScript React template?

Use the React TypeScript template name:

```sh
npm create vite@latest react-learning-lab -- --template react-ts
```

Then install the project dependencies and run its development script as usual.

### 15. What does `npm install` do here?

It reads the project's dependency manifest and installs the packages needed by the starter. It also creates or updates the package lockfile that records the resolved package versions.

### 16. What does `npm run dev` do?

It starts the project's development server and prints a local address. Open that address in a browser. Leave the command running while you work; press Ctrl+C in that terminal to stop it.

### 17. Is the development server the deployed website?

No. It is a local authoring server. Deployment requires building the app and publishing its assets or configuring the framework and hosting provider.

### 18. What are `npm run build` and `npm run preview` for in a Vite starter?

The build script creates production assets. The preview script serves those built assets locally so you can inspect the production build. Preview does not publish the app or configure hosting.

### 19. Should I commit the package lockfile?

Yes. Commit the lockfile created by your package manager so future installs can use the dependency versions recorded for the project.

### 20. Is Create React App still the recommended way to start?

No. Create React App has been deprecated. Do not begin a new project using old instructions that run `npx create-react-app`; choose maintained setup instructions that fit your goal.

## Troubleshooting and safe choices

### 21. Why does a plain Vite React project not show routes automatically?

A minimal Vite starter is a build-tool project, not a full application framework. If the app needs routing, choose a framework or add a router intentionally and configure it for the app.

### 22. Why does Vite say my Node version is unsupported?

The Vite version being installed has a minimum Node.js requirement. Check the message, install a supported Node.js release, open a fresh terminal if necessary, and retry. Do not work around an unsupported runtime by copying random commands from an old tutorial.

### 23. Can I put API secrets in frontend environment variables?

Do not put private credentials in browser-delivered code. Frontend variables used by the app can be included in the build and inspected by visitors. Keep secrets on a server-side system and have the frontend call an appropriately protected endpoint.

### 24. How should I choose if I am not sure yet?

Write down the immediate requirements: browser or native UI, number of routes, data-loading needs, rendering needs, existing infrastructure, and team familiarity. For a small learning exercise, use the course's Vite setup. For a new full web app, evaluate a framework before creating the project. For an existing site, investigate adding React in one area.

## Chapter navigation

| Previous chapter | Lesson | Example | Next chapter |
| --- | --- | --- | --- |
| [004 - JavaScript and browser prerequisites](../lessons/004-javascript-and-browser-prerequisites.md) | [Open lesson](../lessons/005-choose-a-react-project-setup.md) | [Open Example](../examples/005-choose-a-react-project-setup.md) | [006 - Project files, scripts, and the development server](../lessons/006-project-files-scripts-and-dev-server.md) |
