# Optimistic UI with useOptimistic — Example

## Goal

Build a like counter that updates immediately while a simulated request runs. On success, the confirmed count becomes the base state. On failure, the temporary count rolls back and an accessible error appears.

This example uses React 19. The fake server is only an in-memory variable in the browser; it does not send or persist data.

## Create a React project

Use a React 19 project. With a standard Vite React starter, install dependencies and run the development server:

    npm create vite@latest optimistic-like-demo -- --template react
    cd optimistic-like-demo
    npm install
    npm run dev

Replace src/App.jsx with the following component.

## src/App.jsx

    import { useOptimistic, useState, useTransition } from "react";

    // A tiny in-memory stand-in for a server.
    // It resets when the page is reloaded.
    let simulatedServerLikes = 12;

    function wait(milliseconds) {
      return new Promise((resolve) => {
        window.setTimeout(resolve, milliseconds);
      });
    }

    async function saveLike(shouldFail) {
      await wait(800);

      if (shouldFail) {
        throw new Error("The simulated save failed.");
      }

      simulatedServerLikes += 1;
      return simulatedServerLikes;
    }

    export default function App() {
      const [likes, setLikes] = useState(12);
      const [optimisticLikes, setOptimisticLikes] = useOptimistic(likes);
      const [isPending, startTransition] = useTransition();
      const [simulateFailure, setSimulateFailure] = useState(false);
      const [message, setMessage] = useState({ type: "", text: "" });

      function handleLike() {
        startTransition(async () => {
          setMessage({ type: "", text: "" });

          // Show the likely result before the simulated request finishes.
          setOptimisticLikes(likes + 1);

          try {
            const confirmedLikes = await saveLike(simulateFailure);

            // State updates after await need their own Transition.
            startTransition(() => {
              setLikes(confirmedLikes);
              setMessage({
                type: "success",
                text: "The simulated server confirmed the like.",
              });
            });
          } catch {
            // The base count was not changed, so the optimistic count
            // returns to it when this Action ends.
            startTransition(() => {
              setMessage({
                type: "error",
                text: "The like was not saved. The count has been restored.",
              });
            });
          }
        });
      }

      return (
        <main>
          <h1>Optimistic like counter</h1>
          <p>
            The count changes immediately. The simulated save takes about
            800 milliseconds.
          </p>

          <p>
            Likes: <output aria-live="polite">{optimisticLikes}</output>
          </p>

          <button type="button" onClick={handleLike} disabled={isPending}>
            {isPending ? "Saving…" : "Like"}
          </button>

          <p>
            <label>
              <input
                type="checkbox"
                checked={simulateFailure}
                onChange={(event) => setSimulateFailure(event.target.checked)}
                disabled={isPending}
              />
              Simulate a failed save
            </label>
          </p>

          {message.type === "error" && <p role="alert">{message.text}</p>}
          {message.type === "success" && <p role="status">{message.text}</p>}
        </main>
      );
    }

## Try it

1. Click Like. The count increases immediately and the button displays Saving….
2. Wait for the confirmation. The simulated server count becomes the confirmed base value.
3. Select Simulate a failed save, then click Like again.
4. The count increases temporarily, the request fails, and the display returns to its previous confirmed count.
5. Turn off the failure option and submit again to see a successful update.

## How the example works

- likes is the confirmed component state and is the base value passed to useOptimistic.
- setOptimisticLikes runs inside the Action created by startTransition.
- isPending disables repeated clicks while this one Action is active.
- saveLike stands in for an asynchronous request and returns the confirmed count.
- After awaiting, a nested startTransition updates the base likes value.
- If the request throws, the catch block leaves likes unchanged. The temporary optimistic value ends with the Action, restoring the old count.
- The message explains whether the operation succeeded or rolled back.

In a real application, replace saveLike with the application's request or server-supported Action. Validate and authorize the operation on the server, and use the server's response as the confirmed value. Client-side optimistic state is never proof that a change was accepted.
