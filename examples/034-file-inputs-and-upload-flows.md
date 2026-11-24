# 034. File inputs and upload flows - Example

This example lets a person choose up to three PNG or JPEG files, reviews their names and sizes, checks a few client-side limits, and sends a multipart request to an application endpoint. The file input stays browser-managed; React stores the selected File objects only to render the review list.

## Project files

Create these files in a React app created with Vite:

- src/App.jsx
- src/main.jsx
- src/index.css

## src/App.jsx

~~~jsx
import { useRef, useState } from 'react';

const maxFiles = 3;
const maxBytes = 5 * 1024 * 1024;
const allowedTypes = ['image/jpeg', 'image/png'];

function validateFiles(files) {
  const errors = [];

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

    if (!file.type) {
      errors.push(file.name + ': the browser did not report a file type.');
    } else if (!allowedTypes.includes(file.type)) {
      errors.push(file.name + ' is not reported as a PNG or JPEG image.');
    }
  }

  return errors;
}

function App() {
  const fileInputRef = useRef(null);
  const [files, setFiles] = useState([]);
  const [issues, setIssues] = useState([]);
  const [uploading, setUploading] = useState(false);
  const [status, setStatus] = useState('');

  function handleFileChange(event) {
    const selectedFiles = Array.from(event.currentTarget.files ?? []);

    setFiles(selectedFiles);
    setIssues(selectedFiles.length > 0 ? validateFiles(selectedFiles) : []);
    setStatus('');
  }

  function clearSelection() {
    if (fileInputRef.current) {
      fileInputRef.current.value = '';
    }

    setFiles([]);
    setIssues([]);
    setStatus('');
  }

  async function handleSubmit(event) {
    event.preventDefault();

    const form = event.currentTarget;
    const nextIssues = validateFiles(files);

    setIssues(nextIssues);
    setStatus('');

    if (nextIssues.length > 0) {
      return;
    }

    setUploading(true);
    setStatus('Uploading selected filesâ€¦');

    try {
      const response = await fetch('/api/uploads', {
        method: 'POST',
        body: new FormData(form),
      });

      if (!response.ok) {
        throw new Error('The server did not accept the upload.');
      }

      if (fileInputRef.current) {
        fileInputRef.current.value = '';
      }

      setFiles([]);
      setIssues([]);
      setStatus('Upload complete.');
    } catch {
      setStatus('Upload failed. Check the connection and try again.');
    } finally {
      setUploading(false);
    }
  }

  return (
    <main className="page">
      <h1>Upload images</h1>
      <p>
        Choose up to three PNG or JPEG images. Each file must be 5 MB or
        smaller. The server endpoint must be connected for the upload request
        to succeed.
      </p>

      <form onSubmit={handleSubmit}>
        <div className="field">
          <label htmlFor="attachments">Images</label>
          <p id="file-help">
            Allowed formats: PNG and JPEG. Maximum three files, 5 MB each.
          </p>
          <input
            ref={fileInputRef}
            id="attachments"
            name="attachments"
            type="file"
            accept="image/png,image/jpeg,.png,.jpg,.jpeg"
            multiple
            disabled={uploading}
            onChange={handleFileChange}
            aria-invalid={issues.length > 0 ? 'true' : undefined}
            aria-describedby={
              issues.length > 0 ? 'file-help file-errors' : 'file-help'
            }
          />
        </div>

        {issues.length > 0 && (
          <div className="errors" id="file-errors" role="alert">
            <h2>Review the file selection</h2>
            <ul>
              {issues.map((issue, index) => (
                <li key={issue + '-' + index}>{issue}</li>
              ))}
            </ul>
          </div>
        )}

        {files.length > 0 && (
          <section className="selection" aria-labelledby="selection-title">
            <h2 id="selection-title">Selected files</h2>
            <ul>
              {files.map((file, index) => (
                <li key={file.name + '-' + file.lastModified + '-' + index}>
                  <strong>{file.name}</strong>
                  <span>
                    {Math.ceil(file.size / 1024)} KB
                    {file.type ? ' Â· ' + file.type : ' Â· Type not reported'}
                  </span>
                </li>
              ))}
            </ul>
          </section>
        )}

        <div className="actions">
          <button type="submit" disabled={uploading}>
            {uploading ? 'Uploadingâ€¦' : 'Upload selected files'}
          </button>
          <button
            type="button"
            className="secondary"
            onClick={clearSelection}
            disabled={uploading}
          >
            Clear selection
          </button>
        </div>
      </form>

      {status && <p className="status" role={status.startsWith('Upload failed') ? 'alert' : 'status'}>{status}</p>}
    </main>
  );
}

export default App;
~~~

## src/main.jsx

~~~jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './index.css';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

## src/index.css

~~~css
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  color: #172033;
  background: #f4f6fa;
  font-family: system-ui, sans-serif;
  line-height: 1.5;
}

button,
input {
  font: inherit;
}

.page {
  width: min(100% - 2rem, 46rem);
  margin: 3rem auto;
  padding: 2rem;
  background: #fff;
  border: 1px solid #c7cfdb;
  border-radius: 0.75rem;
}

h1,
h2 {
  line-height: 1.2;
}

.field {
  display: grid;
  gap: 0.5rem;
  margin-block: 1.5rem;
}

.field label {
  font-weight: 700;
}

input[type='file'] {
  width: 100%;
  padding: 0.75rem;
  border: 1px solid #667085;
  border-radius: 0.35rem;
  background: #fff;
}

input:focus-visible,
button:focus-visible {
  outline: 3px solid #2459d3;
  outline-offset: 2px;
}

.selection,
.errors {
  margin-block: 1.5rem;
  padding: 1rem 1.25rem;
  border-radius: 0.4rem;
}

.selection {
  background: #f8f9fc;
  border: 1px solid #c7cfdb;
}

.errors {
  color: #551313;
  background: #fff1f0;
  border: 2px solid #9c1c1c;
}

.selection h2,
.errors h2 {
  margin-top: 0;
}

.selection ul,
.errors ul {
  padding-left: 1.5rem;
}

.selection li + li,
.errors li + li {
  margin-top: 0.5rem;
}

.selection li span {
  display: block;
  color: #475467;
}

.actions {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-top: 1.5rem;
}

button {
  padding: 0.7rem 1rem;
  color: #fff;
  background: #1747a6;
  border: 0;
  border-radius: 0.35rem;
  cursor: pointer;
}

button:hover:not(:disabled) {
  background: #103778;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.65;
}

button.secondary {
  color: #172033;
  background: #e4e8ef;
  border: 1px solid #8a94a6;
}

.status {
  margin-top: 1.5rem;
  padding: 1rem;
  background: #e9f7ed;
  border: 1px solid #2b6a3f;
  border-radius: 0.35rem;
}

@media (max-width: 36rem) {
  .page {
    margin-block: 1rem;
    padding: 1.25rem;
  }
}
~~~

## How the example works

- The file input has no value prop. The browser owns the selected files.
- The change handler copies the FileList into an array so React can render selected names, sizes, and reported types.
- The client checks for at least one file, at most three files, a maximum size of 5 MB, and reported PNG or JPEG types.
- These checks are for feedback only. A file's reported type can be missing or untrustworthy, so the server must enforce its own upload policy.
- The submit handler creates FormData from the form and posts it to /api/uploads. It does not set Content-Type; the browser supplies the multipart boundary.
- Pending, success, and failure messages are rendered separately from the file input.
- The clear action empties the DOM input through a ref and clears the matching React state.

## Connect the upload endpoint

This Vite example expects an application endpoint at /api/uploads that accepts a multipart POST request with one or more files named attachments. Without that backend route, selection and client checks still work, but the request will fail and the selected files remain available for retry.

A real endpoint should authenticate and authorize the request, enforce file-count and size limits, validate received content instead of trusting filenames or browser MIME types, store files safely, and return a success response only after the upload operation completes.

## Experiments

1. Choose one or more PNG or JPEG files. Confirm that the selection list shows names, sizes, and reported types.
2. Choose a file larger than 5 MB or a different type. Confirm that readable errors appear and are associated with the input.
3. Clear the selection, then choose the same file again. Confirm that clearing the DOM input permits a fresh selection.
4. Submit valid files without a backend. Confirm that the pending message appears and the interface reports a failure instead of claiming success.
5. Connect a local endpoint at /api/uploads and confirm that a successful server response clears the input and file list.
6. Make the endpoint return a non-success response and confirm the files remain available for retry.
7. Use only a keyboard to reach the file picker and both buttons.

## Important limitation

Client-side metadata and checks do not prove that a file is safe or valid. This example's endpoint is not included; real uploads require server-side validation, authorization, size limits, and safe storage.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [033 - Uncontrolled forms and refs](../lessons/033-uncontrolled-forms-and-refs.md) | [034 - File inputs and upload flows](../lessons/034-file-inputs-and-upload-flows.md) | [Open Q&A](../qnas/034-file-inputs-and-upload-flows.md) | [035 - Browser APIs and event listeners](../lessons/035-browser-apis-and-event-listeners.md) |
