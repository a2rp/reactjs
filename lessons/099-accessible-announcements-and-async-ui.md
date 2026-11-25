# Accessible Announcements and Asynchronous Interfaces in React

## Learning goals

After this lesson, you should be able to:

- Explain why visual updates may need a corresponding accessible announcement.
- Choose a polite status, an urgent alert, or no live announcement based on the update.
- Keep a live region in the rendered page before its text changes.
- Expose loading, success, empty, and error states to sighted and screen-reader users.
- Use aria-busy while a region is being updated.
- Prevent older asynchronous responses from overwriting newer results.
- Avoid repeated, noisy, or duplicate announcements.

## 1. Async changes are not automatically heard

In a traditional page load, a user can start reading the newly loaded document from its beginning. In a React application, a network request may update one small part of the screen while keyboard focus and the screen reader's reading position stay elsewhere.

A sighted user may notice a spinner, a result count, or an error message. A screen-reader user may not notice that the content changed unless the interface exposes the important update. The page needs visible feedback and, when appropriate, a programmatic announcement.

Do not move focus for every update. A status region can communicate progress while allowing the user to keep reading or typing.

## 2. Choose the announcement level

Use the least interruptive mechanism that still informs the user:

| Update | Typical approach |
| --- | --- |
| Routine save confirmation, search count, or background refresh | A status region, usually polite |
| A time-sensitive error that needs immediate attention | An alert, sparingly |
| Long-running measurable work | Visible progress plus a progress indicator with its current value |
| Content the user can find by navigating to it and which is not important immediately | No live announcement may be appropriate |

The status role represents advisory information. It is implicitly polite: assistive technology can announce it at a suitable pause without interrupting current speech.

The alert role represents urgent information and is assertive. It may interrupt what the person is listening to. Do not use it for every validation note, save completion, or ordinary search result.

Do not assign a live region to every changing element. Too many announcements compete with the user's task.

## 3. Keep a live region ready for updates

A robust pattern is to render an empty status container before an async event occurs, then update its text:

    function SaveStatus({ message }: { message: string }) {
      return <p role="status">{message}</p>;
    }

The component stays mounted while message changes. The browser's accessibility layer can detect the change and announce it. A live region generally does not need focus.

Some assistive technologies may miss a live region that appears at the same moment as its message. Keeping the region present and initially empty before the update is more reliable.

Use a short, useful message. For example, “3 results for buttons” tells the user what changed. Repeating the entire result list in the status can be overwhelming; the user can navigate to the list when ready.

## 4. Keep visible status and live status in sync

A message that helps screen readers should also be visible to sighted users. Show loading and completion text in the page rather than putting meaningful feedback in a visually hidden-only region.

A visually hidden live region can be appropriate when the visible interface already presents equivalent information but the dynamic change itself otherwise would not be announced. Do not hide information that other users need.

For a status that is already visible, one element can provide both the visual message and the announcement:

    <p role="status">{statusMessage}</p>

The role supplies the polite status semantics. Avoid piling on redundant ARIA attributes without a reason; duplicate live-region signaling can lead to repeated speech in some combinations of browsers and assistive technology.

## 5. Report loading and progress clearly

When work starts, show a useful text state such as “Searching…” or “Saving changes…”. When it finishes, replace it with a result such as “No matching products” or “Changes saved.”

A spinner alone does not explain what is happening. If the spinner adds no information beyond adjacent text, hide it from assistive technology. If it communicates meaningful progress, provide an accessible name and state.

Use a progressbar only when there is a real progress value to report. For determinate work, provide a current value and suitable minimum and maximum values. For indeterminate work, do not invent a percentage. Keep a text status available so the purpose is clear.

A busy region can be marked with aria-busy="true" while several related parts are being updated. Set it back to false when the coherent update is ready. aria-busy signals that a region is incomplete; it does not by itself tell the user what operation is running, so provide a status message too.

Mark the smallest meaningful region as busy. Do not mark the entire application busy for a small search result update, and do not leave a region busy after success or failure.

## 6. Handle errors without losing context

Make an error visible, specific, and useful. Explain what happened and what the user can do next. For a failed request, a concise message such as “Search could not be completed. Try again.” is more useful than “Error.”

Use an alert only when immediate interruption is warranted. A routine form or request error can often be presented with a polite status plus nearby explanatory text. For field-specific validation, associate the message with the field and expose its invalid state. Keep the label and correction instructions understandable.

Do not announce the same failure through several live regions at once. A single summary or field message is usually easier to follow than an alert, a status, and a toast all repeating the same sentence.

## 7. Avoid announcement noise

Announce meaningful milestones, not every implementation detail:

- Do not announce search results on every keystroke unless the interaction specifically requires it.
- Debounce noisy updates or announce only after an explicit submit or a pause.
- Do not announce every item in a large result list; announce a concise count.
- Do not repeatedly announce an unchanged message.
- Avoid sending progress updates so often that they interrupt reading.
- Keep alert messages short and reserve them for urgent cases.
- Keep the status text stable while the user navigates the rest of the page.

If a result list changes, announce a summary and let users navigate to the list. Do not move focus to the list after every result unless the workflow calls for it.

## 8. Async requests can finish out of order

Suppose a user submits a search for “button” and then quickly submits “dialog.” The second request may finish first. If the old request later overwrites the new results, the interface shows an answer for the wrong query and announces confusing feedback.

Possible solutions include aborting the previous request or assigning each request an identifier and ignoring responses that are no longer current. The UI should reflect the latest user intent.

When a component unmounts, avoid applying results to a screen that no longer exists. Use cleanup or cancellation as appropriate for the data API.

React can manage pending state for forms and transitions, but the application still decides what messages are useful and how users are informed. An isPending boolean is not an accessible announcement by itself.

## 9. React state and a persistent status region

Keep the status region in the component tree and change its text as state changes:

    function ResultsStatus({ isLoading, count }: {
      isLoading: boolean;
      count: number;
    }) {
      let message = '';

      if (isLoading) {
        message = 'Loading results.';
      } else if (count === 0) {
        message = 'No matching results.';
      } else {
        message = count + ' results found.';
      }

      return <p role="status">{message}</p>;
    }

The visible status can be separate from a results region marked busy. That lets the user hear that a search started while assistive technology understands that the result content is still being updated.

## 10. Test the whole async interaction

Test with slow, successful, empty, and failed responses. Verify the visual feedback and announcement for each state. Repeat submissions quickly to confirm stale responses do not replace the latest result.

Check with a screen reader that:

- The status exists before its text changes.
- Loading and completion messages are concise and useful.
- The result list can be found through normal navigation.
- Alerts do not fire for routine updates.
- Focus remains where the user expects.
- Errors identify a recovery action.
- Busy state clears on success and failure.

ARIA behavior varies somewhat across browser and assistive technology combinations. Manual testing is necessary in the environments that matter to the application.

## Common mistakes

- Updating visible content without telling assistive technology when the update matters.
- Creating a live region and its announcement in one simultaneous render.
- Moving focus to every new result.
- Using assertive alerts for ordinary updates.
- Announcing each character typed or every result item.
- Hiding meaningful status from sighted users.
- Treating aria-busy as the announcement text.
- Leaving aria-busy true after an operation ends.
- Repeating the same update through multiple alerts and status regions.
- Letting an older response replace newer results.
- Assuming React's pending state announces itself to screen readers.

## Summary

Async React interfaces need clear feedback for all users. Keep a concise status region ready before it changes, use polite announcements for routine updates, reserve alerts for urgent messages, and mark only incomplete content as busy. Keep focus stable, handle stale responses, and test announcements with real browser and assistive technology combinations.

## Practice checklist

1. Add a persistent status message to a search page.
2. Show and announce loading, results, and empty states.
3. Add an error message with a clear next step.
4. Mark a partially updated region busy, then clear the busy state.
5. Submit two searches quickly and ensure only the latest response is presented.
6. Use a screen reader to confirm that the message is useful and not repetitive.
