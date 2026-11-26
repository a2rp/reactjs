# 034. File inputs and upload flows

## Goal

A file input lets the person choose local files for an application to process. The browser exposes the chosen files as File objects through the input's FileList; React does not control a file input's selected path the way it controls text.

This chapter covers selecting and inspecting files, simple client-side checks, clearing a selection, and sending files as multipart form data. A client-side check improves the experience, but the server must validate every uploaded file independently.

## Learning outcomes

After this chapter, you should be able to:

- Create a labeled file input and read its selected File objects.
- Handle one file or multiple files through the input's files property.
- Explain what accept can and cannot enforce.
- Show a useful selection summary and let a person clear the selection.
- Build a FormData body for an upload request.
- Handle pending, success, and failure states.
- Explain why filename, MIME type, and client-side validation cannot be trusted by the server.

## A file input is browser-managed

Render a native file input with a meaningful label and name:

~~~jsx
<label htmlFor="attachments">Choose images</label>
<input
  id="attachments"
  name="attachments"
  type="file"
  accept="image/png,image/jpeg"
/>
~~~

After a person makes a selection, read the FileList from the input's files property:

~~~jsx
function handleFileChange(event) {
  const selectedFiles = Array.from(event.currentTarget.files ?? []);
  console.log(selectedFiles);
}
~~~

Each File represents a file made available by the user. It provides metadata such as its name, size in bytes, MIME type, and last-modified time, along with the file's contents through the browser APIs.

Do not pass a string value to a file input to try to set a selected file. Browsers do not allow a page to select an arbitrary local path for the person. In React, treat file inputs as uncontrolled; read their files property in an event handler or read them from FormData when the form is submitted.

## Accept one or several files

A file input allows one file by default. Add multiple to allow the person to choose more than one:

~~~jsx
<input
  id="attachments"
  name="attachments"
  type="file"
  accept="image/png,image/jpeg"
  multiple
/>
~~~

When multiple is present, files contains all selected File objects. Convert the FileList to an array if you want to map over it or store a stable selection in React state:

~~~jsx
const selectedFiles = Array.from(event.currentTarget.files ?? []);
setFiles(selectedFiles);
~~~

The accept attribute is a picker hint, not a security check. It can make the file chooser favor certain file types, but the application must still validate the selected files and the server must check uploaded data independently. A file's name or type reported by the client is not proof of what its contents are.

## Display a useful selection summary

When showing chosen files in React, store the selected File objects in state. This state drives the preview of names and sizes; it does not control the native file input's selected path:

~~~jsx
const [files, setFiles] = useState([]);

function handleFileChange(event) {
  setFiles(Array.from(event.currentTarget.files ?? []));
}

return (
  <>
    <label htmlFor="attachments">Choose up to three images</label>
    <input
      id="attachments"
      name="attachments"
      type="file"
      accept="image/png,image/jpeg"
      multiple
      onChange={handleFileChange}
    />

    <ul>
      {files.map((file) => (
        <li key={file.name + file.lastModified}>
          {file.name} - {Math.ceil(file.size / 1024)} KB
        </li>
      ))}
    </ul>
  </>
);
~~~

A file can have an empty MIME type, and the same filename can be selected more than once from different places. Do not use a filename alone as a guaranteed unique identifier. This example combines the name and last-modified time for a simple display key; production lists should handle possible collisions.

If you generate image previews with URL.createObjectURL, revoke each object URL with URL.revokeObjectURL when it is no longer needed. A filename and size list avoids creating preview URLs when they are not necessary.

## Add client-side checks for a better experience

A client can check file count, size, and reported type before sending. Explain every limit near the control and show a readable error:

~~~jsx
function validateFiles(files) {
  const errors = [];
  const maxFiles = 3;
  const maxBytes = 5 * 1024 * 1024;
  const allowedTypes = ['image/jpeg', 'image/png'];

  if (files.length === 0) {
    errors.push('Choose at least one image.');
  }

  if (files.length > maxFiles) {
    errors.push('Choose no more than three images.');
  }

  for (const file of files) {
    if (file.size > maxBytes) {
      errors.push(file.name + ' is larger than 5 MB.');
    }

    if (!allowedTypes.includes(file.type)) {
      errors.push(file.name + ' is not reported as a PNG or JPEG image.');
    }
  }

  return errors;
}
~~~

Use text, not only a colored border or icon, to explain a problem. Associate instructions and errors with the file input using a visible label and aria-describedby where needed. Present dynamic results accessibly; the previous chapter covers error messaging patterns.

These checks are helpful feedback, not a security boundary. The person can bypass or alter browser code, file metadata can be missing or misleading, and requests can be sent without this interface. The server must enforce count and size limits, inspect and validate the received content, authorize the upload, and store it safely.

## Clear a file selection

The browser does not let a page fill the file input with a local file path. It does allow the page to clear the input by assigning an empty string to its value. A ref can perform that focused DOM action:

~~~jsx
const fileInputRef = useRef(null);

function clearSelection() {
  if (fileInputRef.current) {
    fileInputRef.current.value = '';
  }

  setFiles([]);
  setErrors([]);
}
~~~

Use the ref in JSX and offer a button with an explicit non-submit type:

~~~jsx
<input ref={fileInputRef} type="file" name="attachments" multiple />
<button type="button" onClick={clearSelection}>
  Clear selected files
</button>
~~~

Clear both the DOM input and any React state that displays the selection. If you clear only the state, the file input still contains the selected files and a later submission can include them. If you clear only the input, the old file names can remain in the rendered list.

The browser's native form reset also clears the selected files. If the form has other React state, reset that state separately.

## Read files from FormData

A named file input is included in FormData. A single-file input contributes one File value; a multiple-file input can contribute multiple values under the same name:

~~~jsx
const formData = new FormData(event.currentTarget);
const attachments = formData.getAll('attachments');
~~~

Use getAll for a repeated name. For a native file input, this gives you File objects for the selected files. You can also append File objects yourself when creating a new FormData instance.

## Send multipart form data

A browser can send FormData as the body of a fetch request:

~~~jsx
async function upload(form) {
  const formData = new FormData(form);

  const response = await fetch('/api/uploads', {
    method: 'POST',
    body: formData,
  });

  if (!response.ok) {
    throw new Error('The upload could not be completed.');
  }

  return response;
}
~~~

Do not manually set the Content-Type header to multipart/form-data when passing FormData to fetch. The browser must add the multipart boundary that separates the fields and file parts. Setting the header yourself without that boundary can make the body impossible for the server to parse.

The endpoint is an application server route, not a built-in React feature. It must accept multipart data, enforce server-side limits and validation, and return a suitable response. A successful client request does not mean the server stored the file permanently or safely; follow the server's response contract.

## Model an asynchronous upload

An upload interface usually needs to represent at least three states: idle, uploading, and a result or error. Disable duplicate submission while the request is in progress, keep a clear status message, and let the user retry if the request fails.

~~~jsx
const [uploading, setUploading] = useState(false);
const [status, setStatus] = useState('');

async function handleSubmit(event) {
  event.preventDefault();
  setStatus('');
  setUploading(true);

  try {
    const response = await fetch('/api/uploads', {
      method: 'POST',
      body: new FormData(event.currentTarget),
    });

    if (!response.ok) {
      throw new Error('The upload failed. Try again.');
    }

    setStatus('Upload complete.');
  } catch (error) {
    setStatus(error.message || 'The upload failed. Try again.');
  } finally {
    setUploading(false);
  }
}
~~~

In a real component, capture the form element before awaiting if you need to use it again after the request. Network work can take time, and the submit event's currentTarget is only available while its handler is running.

A status message can use role=status for non-urgent progress or completion feedback. If an upload is canceled or fails, explain what happened and what can be tried next. Do not announce success until the server confirms success.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Passing a path or selected file string as the input value | Browsers do not let a page choose an arbitrary local file. | Read user-selected files from the input's files property. |
| Treating a file input like a controlled text input | The input's selected file list is browser-managed. | Keep the file input uncontrolled and store File objects only if the UI needs them. |
| Assuming accept enforces the allowed types | It only guides the picker and can be bypassed. | Validate in the client for feedback and enforce rules again on the server. |
| Trusting the filename or reported MIME type | Client-provided metadata can be missing or misleading. | Inspect and validate the received content on the server. |
| Using get when multiple files share one name | get returns only one matching value. | Use getAll for a multiple-file field. |
| Clearing React's file list but not the input element | The old DOM selection can still be submitted. | Clear the input through a ref and clear matching UI state. |
| Setting the multipart Content-Type header manually | The browser may not add the boundary needed to parse the request. | Pass FormData as the body and let the browser set the header. |
| Showing “Upload complete” before the server responds | A pending or failed request is reported as successful. | Show success only after checking the response. |
| Sending files without server-side limits or authorization | Browser checks can be bypassed and large or unsafe content can reach storage. | Enforce size, type, count, access, and storage rules on the server. |

## Practice

1. Add a labeled file input that accepts PNG and JPEG images.
2. Allow multiple files and display each selected file's name and size.
3. Add a client-side maximum file count and size check with visible text errors.
4. Add a clear-selection button that clears both the input through a ref and the displayed file list.
5. Build FormData from a form with a multiple-file input and read its values with getAll.
6. Add a fetch request using FormData without manually setting Content-Type.
7. Add idle, uploading, success, and failure messages, and prevent duplicate submits while uploading.
8. Write down the server checks required even when the client validates count, size, and type.
9. If you make image previews, revoke every object URL when its preview is removed or the component is cleaned up.

## Key takeaways

- A file input is browser-managed; read selected File objects from its files property.
- Use multiple for several files and getAll when reading repeated names from FormData.
- accept is a picker hint, not validation or security.
- Use React state to display file metadata when useful, but do not use it to control the selected path.
- Clear both the DOM file input and any React state that mirrors its selection.
- Send uploads as FormData and let the browser set the multipart boundary.
- Validate, authorize, limit, and safely store uploaded files on the server.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="033-uncontrolled-forms-and-refs.md">033 - Uncontrolled forms and refs</a></td><td><a href="../qnas/034-file-inputs-and-upload-flows.md">Q&amp;A</a></td><td><a href="../examples/034-file-inputs-and-upload-flows.md">Example</a></td><td><a href="035-browser-apis-and-event-listeners.md">035 - Working with browser APIs and external event listeners</a></td></tr>
  </tbody>
</table>
