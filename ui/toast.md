---
title: "Toast Notifications"
description: "Lightweight status messages displayed at the top-centre of the viewport. Toasts auto-dismiss after a configurable duration and are announced to screen..."
---
Lightweight status messages displayed at the top-centre of the viewport. Toasts auto-dismiss after a configurable duration and are announced to screen readers via `aria-live`.

There are two ways to trigger a toast: a declarative HTML attribute for immediate feedback on click, and post-action attributes on forms/buttons that fire after a REST or API operation completes.

---

## 1. Immediate Toast on Click (`data-toast`)

Add `data-toast` to any element. Clicking it shows the toast immediately — no operation required. Useful for "coming soon" links or non-destructive inline feedback.

### Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-toast` | Yes | — | Message to display |
| `data-toast-variant` | No | `"warning"` | `"info"`, `"success"`, `"warning"`, or `"error"` |

### Examples

```html
<!-- "Coming soon" link that won't navigate -->
<a href="#" data-toast="This feature is coming soon." data-toast-variant="info">
  Reports
</a>

<!-- Inline success feedback -->
<button data-toast="Copied to clipboard!" data-toast-variant="success">
  Copy
</button>

<!-- Default variant is "warning" -->
<button data-toast="This action cannot be undone.">
  Archive
</button>
```

Clicking an `<a>` with `data-toast` prevents default navigation. Clicking any other element triggers the toast without interfering with the element's normal behaviour.

---

## 2. Post-Action Toast (`data-toast-success` / `data-toast-error`)

Add these attributes to a `<form>` or `<button>` that is handled by [RestController](/controllers/rest-controller) or [ApiController](/controllers/api-controller). The toast fires after the operation resolves.

### Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-toast-success` | No | — | Message shown when the operation succeeds |
| `data-toast-error` | No | `err.message` | Message shown when the operation fails. Falls back to the server error message if omitted. |

### Examples

```html
<!-- Form with success and error toasts -->
<form data-rest-content="load"
      data-toast-success="Saved successfully."
      data-toast-error="Save failed — please try again.">
  <input name="title">
  <button type="submit">Save</button>
</form>

<!-- Button operation — only success toast, rely on default error message -->
<button data-rest-content="delete"
        data-content-params='{"contentId":"abc-123"}'
        data-toast-success="Deleted.">
  Delete
</button>

<!-- API operation -->
<form data-api-url="/api/publish" data-api-method="POST"
      data-toast-success="Published!"
      data-toast-error="Publish failed.">
  <input type="hidden" name="id" value="...">
  <button type="submit">Publish</button>
</form>
```

---

## 3. JavaScript API

Call `showToast()` directly for programmatic use.

```js
import { showToast } from "./ui/toast.js";

// Basic usage
showToast("Item saved.");

// With variant
showToast("Something went wrong.", { variant: "error" });

// Custom duration (ms) — 0 means no auto-dismiss
showToast("Upload in progress...", { variant: "info", duration: 0 });
```

### `showToast(message, options)`

| Parameter | Type | Default | Description |
|---|---|---|---|
| `message` | string | — | Text to display |
| `options.variant` | string | `"info"` | `"info"`, `"success"`, `"warning"`, or `"error"` |
| `options.duration` | number | `4000` | Milliseconds before auto-dismiss. `0` disables auto-dismiss. |

Returns the toast DOM element.

### `dismiss(toast)`

Immediately removes a specific toast element:

```js
const toast = showToast("Uploading...", { duration: 0 });
// later...
dismiss(toast);
```

---

## Variants

| Variant | Use for |
|---|---|
| `info` | Neutral status updates, progress messages |
| `success` | Completed operations |
| `warning` | Non-blocking caution messages |
| `error` | Failed operations or validation errors |
