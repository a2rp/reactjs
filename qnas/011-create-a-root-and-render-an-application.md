# 011. Create a root and render a React application - Q&A

## The root container and startup

### 1. What is a React root container?

It is a real browser DOM element where React manages a rendered component tree. In a simple Vite app, the HTML document commonly contains an empty element with the id root.

### 2. Is the id root required?

No. It is a common convention. The id in the HTML and the value used by the JavaScript lookup just need to match.

### 3. Where should createRoot be imported from?

Import createRoot from react-dom/client. It is a React DOM client API.

### 4. What does createRoot do?

It creates a React root associated with the DOM container. The root can then render a React node inside that container.

### 5. What does root.render do?

It displays the React node passed to it, usually the App component, inside the root container. The call to createRoot sets up React's ownership; render displays the component tree.

### 6. Why should the entry module check that the container exists?

document.getElementById can return null when the id does not exist or does not match. Passing a missing container to createRoot causes startup to fail. A clear check helps identify the mismatch.

### 7. Should I call App() directly?

No. Render the component as a React element through the root. Calling the component function yourself bypasses React's normal component management and can break features such as Hooks.

### 8. How many roots does a typical fully React-built app need?

Usually one root for the whole app. Components inside the root are composed under the app component; they do not each need their own root.

### 9. Can one page have more than one React root?

Yes. A page that gradually adds React can create separate roots for separate DOM regions. Each root should have its own container. A typical app built fully with React usually needs only one.

### 10. Should createRoot be called from inside a component?

No. Create the root once in the application entry module. Calling it from a component could create competing roots during normal rendering instead of composing components in one tree.

## Rendering and updates

### 11. Does the first client render preserve HTML already inside the root container?

No. The first call to root.render clears the existing content inside that container and renders the React tree there. Keep page content that must stay static outside the root element.

### 12. Does root.render clear the entire HTML page?

No. It manages the contents of the DOM element passed to createRoot. Siblings outside that container remain outside React's managed region.

### 13. Should I call root.render every time some data changes?

Usually not. The entry module normally creates the root and renders the app once. Components describe later UI updates using props and state. Calling root.render again on the same root updates the rendered tree, but it is not the normal way to handle every interaction.

### 14. What does root.render return?

It returns undefined. Do not treat its return value as the rendered DOM or as proof that effects have already run.

### 15. What happens if I pass a missing element to createRoot?

There is no DOM node for React to manage, so root creation fails. Check the element lookup and make sure the HTML id is correct before calling createRoot.

### 16. What is the difference between createRoot and hydrateRoot?

Use createRoot for a client-rendered app whose contents will be rendered by the browser. Use hydrateRoot when React already generated the initial HTML on the server and the browser should attach React behavior to that matching HTML.

### 17. Can I use createRoot for server-rendered React HTML?

It is not the right API for that case. The first client render clears existing contents. Use hydrateRoot for React-generated server HTML and ensure the initial client output matches it.

### 18. Does a framework app need this entry code?

Often the framework creates and manages the root itself. Follow the framework's entry conventions and avoid adding a second root unless its instructions call for it.

## Cleanup and older setup code

### 19. When should root.unmount be called?

A standalone React app usually stays mounted for the page's lifetime. Call root.unmount when an external system is about to remove the DOM container React manages, so the component tree can clean up.

### 20. Can I render into the same root after calling root.unmount?

No. That root has been detached. If the same DOM element is used again, create a new root for it after the previous root has been unmounted.

### 21. I found ReactDOM.render in an older tutorial. Should I copy it?

For a current client-rendered React app, use createRoot from react-dom/client and then call root.render. Older tutorials may use legacy startup APIs that do not match the modern root setup.

### 22. What should I check first when the browser is blank?

Check that the HTML contains the expected container id, that the entry module runs after the document element is available, that the id lookup matches, and that the browser Console shows no import or runtime error.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="010-fragments-and-multiple-elements.md">010 - Fragments and returning multiple elements - Q&amp;A</a></td><td><a href="../lessons/011-create-a-root-and-render-an-application.md">Lesson</a></td><td><a href="../examples/011-create-a-root-and-render-an-application.md">Example</a></td><td><a href="012-strict-mode-and-development-checks.md">012 - Strict Mode and development checks - Q&amp;A</a></td></tr>
  </tbody>
</table>
