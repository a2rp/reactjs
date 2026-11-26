# 005. Choose a React project setup

## Learning goals

By the end of this lesson, you should be able to:

- Choose a setup for learning, adding React to a site, building a web app, or building a native app.
- Explain the roles of React, a build tool, and a React framework.
- Create a small local React learning project with Vite.
- Choose JavaScript or TypeScript based on your needs.
- Recognize why Create React App should not be used for new projects.

## React needs a project environment

React is a UI library. It provides components and APIs for describing and updating interfaces. A project also needs tooling that can load modules, transform JSX, run a development server, and prepare files for deployment.

- A **build tool** such as Vite serves and transforms code during development and creates production assets. It does not, by itself, decide how the app handles routing or server data.
- A **React framework** provides application conventions and integrated solutions for concerns such as routes, data loading, rendering strategies, and deployment.
- A **browser sandbox** lets you try small React examples without installing a local project.
- **React DOM** renders React UI into a browser document. Native mobile apps use a React Native renderer instead.

These tools can work together. For example, React Router can be used with Vite in framework mode. Their names describe responsibilities, not necessarily separate competing choices.

## Choose a path

| Your situation | Starting point | Reason |
| --- | --- | --- |
| You want to try a small component | An online React sandbox | No local installation is needed. |
| You are learning React fundamentals locally | Vite with the React template | It gives you JSX support and a development server while keeping the project structure visible. |
| You are starting a complete production web app | A React framework such as Next.js or React Router in framework mode | It provides established solutions for routing, data loading, rendering, and deployment. |
| You want to add one interactive area to an existing site | Add React to that site or mount a React root in that area | You can adopt React gradually without rebuilding unrelated pages. |
| You need native Android or iOS UI | Expo with React Native | A browser app setup does not create native mobile screens. |

React recommends starting new apps and websites built fully with React using a framework. Starting from scratch with a build tool remains useful for learning, embedding React in an existing site, or requirements that are not served well by a framework. A plain build-tool starter is generally a client-rendered single-page app; you choose any additional routing, data, and deployment solutions.

A framework does not require every page to use a server. Frameworks can support client rendering and static output as well as server-rendered routes. Choose a rendering strategy based on the app.

## What each layer does

For a page that lists products:

1. **React** describes the interface as components and updates it when inputs change.
2. **React DOM** connects that interface to the browser document.
3. A **build tool** serves the project while developing and transforms source files such as JSX.
4. A **framework**, if selected, can also define how URLs map to pages, where data loads, how pages render, and how deployment works.

React does not include every application feature. You can add a router or data library to a build-tool starter, but you choose and integrate them. A framework makes more of those choices for you.

## When each path fits

### Learning locally

For this course's early browser-based exercises, Vite is a practical teaching environment. It lets you focus on components while the next lesson explains its files, scripts, and development server. This is a learning setup, not a rule that Vite alone is best for every production app.

### Building a full web app

For a new app with several routes, data loading, SEO needs, server rendering, or deployment requirements, evaluate a React framework. Consider hosting, team experience, data flow, and rendering needs. A framework can still deliver client-rendered or static pages.

### Adding React to a current website

If a website already uses another server or page system, mount React only where interactive UI is needed and expand gradually. There is no need to replace stable parts of the site just to adopt React.

### Building for phones

React in a browser is not a native mobile application. For platform-native Android or iOS UI, begin with React Native tooling such as Expo.

## JavaScript or TypeScript?

Both work with React:

- Choose **JavaScript** if you are new to React or want fewer concepts at the start.
- Choose **TypeScript** if your team uses it or you want static checking for props, data, and functions.

TypeScript adds a type system used for development and build checks; it does not change React's component model. This course begins with JavaScript to focus on React. Choose one language consistently in a project and use the matching template.

## Create a local learning project with Vite

Use a supported Node.js release. The current Vite version may set a minimum Node version; if it reports yours as unsupported, install a supported release first.

Check Node.js and npm:

```sh
node --version
npm --version
```

Create and run a JavaScript React starter:

```sh
npm create vite@latest react-learning-lab -- --template react
cd react-learning-lab
npm install
npm run dev
```

Vite prints a local address in the terminal. Open it in a browser and keep the command running while you work. Stop it with Ctrl+C.

For the TypeScript template, use:

```sh
npm create vite@latest react-learning-lab -- --template react-ts
```

If prompted, select React and the language that matches your choice. Prompts may change between releases. The next lesson explains the generated project rather than asking you to memorize every file now.

For a production preview of the Vite starter:

```sh
npm run build
npm run preview
```

Build creates deployable assets; preview serves those assets locally. It does not deploy the app or configure hosting. Commit the package lockfile so installs use the recorded versions. Never put private credentials in browser code because visitors can inspect bundled files.

## Avoid outdated setup instructions

Create React App is deprecated. Do not start a new project with tutorials using `npx create-react-app`. Setup commands and prompts change over time, so use maintained instructions that fit your goal.

## Common mistakes

- **Calling Vite a full-stack framework:** a plain Vite starter does not automatically supply application routing, server data loading, or a backend.
- **Assuming every framework needs a server:** client-rendered and static apps can also use frameworks.
- **Choosing by popularity alone:** check rendering, hosting, team, and application needs.
- **Mixing several starter instructions:** follow one project's file layout and scripts until you understand them.
- **Treating the development server as deployment:** a local address is only for your development environment.
- **Putting secrets in frontend source:** browser-delivered code is visible to users.

## Key takeaways

- React handles UI; React DOM renders browser UI.
- Build tools handle development and bundling; frameworks add application-level structure and solutions.
- React recommends a framework for a new full React app.
- Vite is useful for learning React and other cases where starting from scratch fits.
- Existing sites can adopt React gradually; Expo and React Native target native mobile UI.
- Choose JavaScript or TypeScript to suit the learner and project.
- Create React App is deprecated.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="004-javascript-and-browser-prerequisites.md">004 - JavaScript and browser prerequisites</a></td><td><a href="../qnas/005-choose-a-react-project-setup.md">Q&amp;A</a></td><td><a href="../examples/005-choose-a-react-project-setup.md">Example</a></td><td><a href="006-project-files-scripts-and-dev-server.md">006 - Project files, scripts, and the development server</a></td></tr>
  </tbody>
</table>
