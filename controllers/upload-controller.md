---
title: "UploadController"
description: "Drag-and-drop file upload zone. Mounted on the drop zone element itself (not ). Validates file type, size, and filename pattern, sets accepted files on a..."
---
Drag-and-drop file upload zone. Mounted on the drop zone element itself (not `<body>`). Validates file type, size, and filename pattern, sets accepted files on a hidden `<input type="file">` target, and dispatches events so the enclosing form and RestController pick them up transparently as `multipart/form-data`. Shows an error toast automatically when a file is rejected.

## Setup

Mount the controller on a wrapper `<div>` that serves as the drop zone. Place the hidden `<input type="file">` inside it with the `input` target attribute.

```html
<div data-controller="cba-upload"
     data-cba-upload-accept-value="text/xml,application/xml"
     data-cba-upload-max-size-value="10485760">

  <label for="file-upload">
    <span data-cba-upload-target="display">Upload a file</span>
    <input type="file"
           name="xmlFile"
           id="file-upload"
           data-cba-upload-target="input"
           class="sr-only">
  </label>
  <p>or drag and drop</p>

</div>
```

## Controller Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-cba-upload-accept-value` | No | — | Comma-separated MIME types or extensions to accept (e.g. `"text/xml,application/xml"` or `".xml,.xsd"`). Falls back to the `accept` attribute on the `<input>` if omitted. |
| `data-cba-upload-max-size-value` | No | `0` (no limit) | Maximum file size in bytes (e.g. `10485760` for 10 MB) |
| `data-cba-upload-name-pattern-value` | No | — | Glob pattern the filename must match. `*` matches any characters, `?` matches one character. Case-insensitive. e.g. `"*.xml"` or `"part-?-topic.xml"` |

## Targets

| Target | Required | Description |
|---|---|---|
| `data-cba-upload-target="input"` | Yes | The `<input type="file">` that receives the validated files. The form submits this input as `multipart/form-data`. |
| `data-cba-upload-target="display"` | No | Any element whose `textContent` is replaced with the selected filename after a successful drop or browse. Shows `"3 files selected"` for multi-file. |

## Validation

All three checks are applied in order. The first failure stops checking and triggers the rejection toast and event.

| Check | Attribute | Failure reason |
|---|---|---|
| File type | `data-cba-upload-accept-value` | `"type"` |
| File size | `data-cba-upload-max-size-value` | `"size"` |
| Filename pattern | `data-cba-upload-name-pattern-value` | `"name"` |

When a file is rejected, an error toast is shown automatically with the filename and reason:

- `"doc.pdf" is not an accepted file type.`
- `"large.xml" exceeds the 1 MB size limit.`
- `"report.pdf" does not match the required filename pattern (*.xml).`

## Events

Both events bubble from the controller element.

| Event | Detail | Description |
|---|---|---|
| `cbapp:files-selected` | `{ files: File[], input: HTMLInputElement }` | Valid files were dropped or selected. `files` contains the accepted `File` objects; `input` is the `<input type="file">` target. |
| `cbapp:files-rejected` | `{ files: File[], reason: "type" \| "size" \| "name" }` | Files failed validation. A toast is shown automatically — listen to this event only if you need additional custom behaviour. |

## CSS Hook

The class `drag-over` is added to the controller element while a drag is active over the drop zone. Use it to style the highlight state in your app CSS:

```css
[data-controller="cba-upload"].drag-over {
  @apply bg-blue-50 outline-2 outline-dashed outline-blue-400;
}
```

## Multiple Files

By default only the first dropped file is accepted. To allow multiple files, add the `multiple` attribute to the `<input>`:

```html
<input type="file" name="files[]" multiple data-cba-upload-target="input" class="sr-only">
```

## Integration with RestController

No changes to the REST layer are needed. The controller sets the validated files on the `<input type="file">` target. When the enclosing form is submitted, `new FormData(form)` picks them up automatically and the request is sent as `multipart/form-data`.

```html
<form data-rest-content="load"
      data-toast-success="Upload successful."
      data-refresh="#file-list">

  <input type="hidden" name="contentId" value="...">

  <div data-controller="cba-upload"
       data-cba-upload-accept-value="text/xml,application/xml"
       data-cba-upload-max-size-value="10485760"
       data-cba-upload-name-pattern-value="*.xml">
    <label for="drop-input">
      <span data-cba-upload-target="display">Upload a file</span>
      <input type="file" name="xmlFile" id="drop-input"
             data-cba-upload-target="input" class="sr-only">
    </label>
    <p>or drag and drop</p>
  </div>

  <button type="submit">Upload</button>
</form>
```

## Complete Example

Drop zone inside a drawer form, with display feedback, filename validation, and close-on-success:

```html
<form data-rest-content="load"
      data-toast-success="Upload successful."
      data-refresh="#comp-mat-list">

  <input type="hidden" name="contentId" value="...">
  <input type="hidden" name="srcFile" value="xmlfile">

  <div class="flex justify-center rounded-lg border border-dashed border-gray-900/25 px-6 py-10">
    <div class="text-center">

      <!-- Upload icon -->
      <svg class="mx-auto size-12 text-gray-300" ...></svg>

      <div data-controller="cba-upload"
           data-cba-upload-accept-value="text/xml,application/xml"
           data-cba-upload-max-size-value="1024000"
           data-cba-upload-name-pattern-value="*.xml"
           class="mt-4 flex text-sm text-gray-600">
        <label class="cursor-pointer font-semibold text-blue-600 hover:text-blue-500"
               for="xml-upload">
          <span data-cba-upload-target="display">Upload a file</span>
          <input type="file" name="filePram.xmlfile" id="xml-upload"
                 data-cba-upload-target="input" class="sr-only">
        </label>
        <p class="pl-1">or drag and drop</p>
      </div>

      <p class="text-xs text-gray-500">XML files only, up to 1 MB</p>
    </div>
  </div>

  <div class="mt-4">
    <button type="submit" data-close-drawer>Upload XML</button>
    <button type="reset" data-action="click->cba-drawer#close">Cancel</button>
  </div>
</form>
```
