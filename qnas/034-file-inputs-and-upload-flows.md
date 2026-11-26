# 034. File inputs and upload flows - Q&A

## Selecting files

### 1. How does a person choose a file in a React form?

Use a native input with type file. The browser opens its file chooser and exposes the selected files to the page after the person chooses them.

### 2. How do I read the selected file objects?

Read the input element's files property in an event handler. It is a FileList, which can be converted to an array with Array.from when array methods or React list rendering are needed.

### 3. What information does a File object provide?

It provides metadata such as the filename, size, reported MIME type, and last-modified time, as well as access to the file's contents through browser APIs.

### 4. Can React control a file input like a text input?

No. Do not pass a string value that tries to select a local file. The browser owns the selected file list. Read it from the DOM and store File objects in React state only when the interface needs to display or process them.

### 5. Why can a page not set a local file path in the file input?

The person must choose which local file to make available. Allowing a page to set an arbitrary local path would expose files without that user selection. A page may clear the control, but cannot fill it with a non-empty path.

### 6. What does the accept attribute do?

It hints to the file chooser which kinds of files the interface expects. It can guide the picker, but does not validate the file or guarantee that the selected contents match the requested type.

### 7. Can accept be used as the only file-type check?

No. It can be bypassed and client metadata can be missing or misleading. Use it for picker guidance and check the received content on the server.

### 8. How do I allow multiple selected files?

Add the multiple attribute to the file input. The files property then exposes the selected File objects as a list.

### 9. Does a file input have a meaningful selected value string?

The value string is not the selected file content or a usable local path. Use the files property to access the File objects. Browsers may display a privacy-protected path-like value, which should not be used for file access.

### 10. How can I show selected filenames in React?

Copy the FileList into an array in the input's change handler, store that array in state, then render its metadata. That state mirrors the UI's selection for display; it does not control the file input.

## Client-side checks and clearing

### 11. What client-side file checks are useful?

Checks for required selection, maximum count, maximum size, and reported type can give immediate feedback. Show the limits near the input and explain which file needs attention.

### 12. Are client-side file checks security controls?

No. They can be changed or skipped, and the request can be sent without the page. The server must independently enforce limits, authorization, and content validation.

### 13. Can a File object's type be empty?

Yes. The reported MIME type can be empty or unreliable. Treat it as a helpful hint for the user interface, not proof of the file's actual content.

### 14. Is checking a filename extension enough to validate a file?

No. Extensions and names are client-provided metadata. The server should validate the received content and apply its upload policy.

### 15. How do I clear a file input?

Use a ref to the DOM input and set its value to the empty string. Also clear any React state that displays the chosen files.

### 16. Why clear both the DOM input and the React file list?

They are separate representations. Clearing only the React list can leave the files selected in the DOM and included in the next form submission. Clearing only the DOM can leave stale names rendered in the interface.

### 17. Does a native form reset clear selected files?

Yes. The browser's form reset behavior clears file selections. If the interface also stores upload status, errors, or a receipt in React state, reset that state separately when appropriate.

### 18. Can a page remove one file from the input's FileList by assigning to value?

The page cannot assign an arbitrary non-empty file selection. A simple clear action empties the whole input. If the interface needs per-file removal from a multi-file selection, maintain an application-level selected-file list and build the upload FormData from that list, or use a carefully managed file-input strategy.

### 19. Can the same file be chosen again after clearing?

Yes. Clearing the DOM input makes it possible to select the same file again and still receive a new change event in common browser behavior.

## FormData and upload requests

### 20. How do I read a file input using FormData?

Give the input a name and create FormData from its containing form. Use get for one file or getAll for a multiple-file field with repeated values.

### 21. What type does FormData return for a file field?

A file field contributes a File object. Text fields contribute strings. Handle the returned value according to the kind of control instead of assuming every entry is a string.

### 22. Why should a multiple-file field use getAll?

Multiple selected files share the same form field name. get returns only one matching entry; getAll returns all entries for that name.

### 23. How do I upload files with fetch?

Create a FormData body, include the file objects and any text fields, then pass it as the request body of a POST request to an upload endpoint implemented by your server.

### 24. Should I manually set Content-Type to multipart/form-data?

No. When fetch sends FormData, the browser sets the multipart content type and its boundary. Setting the header manually can omit the boundary the server needs to parse the parts.

### 25. Does React upload files by itself?

No. React renders the form and handles events. The browser APIs provide File and FormData, and the application sends the request to a server endpoint that accepts uploads.

### 26. Is a relative upload path such as /api/uploads a built-in React route?

No. It is an example endpoint name. The server or development proxy must provide that route and define how it validates and stores uploaded files.

### 27. When should the UI show upload success?

Only after the server responds with a successful result that matches the application's contract. A request being sent or a local validation passing does not mean a file was stored.

### 28. What should the UI do while an upload is pending?

Show a clear pending status, prevent accidental duplicate submissions when appropriate, and keep the user informed. Then show the server-confirmed result or an actionable error.

### 29. Should the form discard files after a failed upload?

Usually preserve the selection so the person can retry. Clear it after success only if that is the intended interaction. Keep the UI's list and the file input selection in sync.

### 30. What if the network request fails?

Show that the upload did not complete and offer a safe next step, such as retrying. Do not report success, and avoid clearing the chosen files before the person can retry.

## Previews and security

### 31. How can I show a local image preview?

A browser can create an object URL for a selected File and use that URL as an image source. If you do this, revoke the object URL with URL.revokeObjectURL when it is no longer needed.

### 32. Why should object URLs be revoked?

They keep a browser-managed reference to the file data. Revoke URLs when a preview is removed or replaced so the application does not retain unnecessary references.

### 33. Is a preview URL the same as uploading the file?

No. An object URL allows the current browser page to display the local file. It does not send that file to a server.

### 34. Is the file name safe to use as a storage path?

No. Treat the submitted filename as untrusted input. A server should choose safe storage names and avoid using client-provided path components as filesystem paths.

### 35. What should a server validate for an upload?

It should enforce authentication and authorization, count and size limits, allowed content, and safe storage behavior. It should not trust accept, file names, or MIME types supplied by the client.

### 36. Does a browser's MIME type prove the file's contents?

No. It is reported client metadata. The server should inspect the received content using an appropriate validation strategy for the application.

### 37. Should the server keep an uploaded file at a public URL by default?

Not necessarily. Storage location and access policy depend on the product. Private or sensitive uploads may require authenticated access, retention controls, or other safeguards.

### 38. How do file inputs fit with accessible forms?

Give the input a visible label, explain accepted formats and size limits in text, associate instructions and errors with the control, and make progress and failure feedback available to assistive technology.

### 39. How should I test a file selection flow?

Use the keyboard to reach the file input and buttons, choose allowed and disallowed files, test size and count boundaries, clear and reselect a file, and check pending, success, and failure states. Test the server policy separately.

### 40. What are the most important upload rules?

Use the browser's files property or FormData to access user-selected File objects, treat accept and client checks as guidance, let the browser set multipart boundaries, and enforce upload policy on the server.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="033-uncontrolled-forms-and-refs.md">033 - Uncontrolled forms and refs - Q&amp;A</a></td><td><a href="../lessons/034-file-inputs-and-upload-flows.md">Lesson</a></td><td><a href="../examples/034-file-inputs-and-upload-flows.md">Example</a></td><td><a href="035-browser-apis-and-event-listeners.md">035 - Working with browser APIs and external event listeners - Q&amp;A</a></td></tr>
  </tbody>
</table>
