---
title: "ViewController"
description: "Loads HTML view fragments into destination elements via fetch. Mounted on by the app shell."
---
Loads HTML view fragments into destination elements via fetch. Mounted on `<body>` by the app shell.

Wire triggers with `data-action="cba-view#load"` on any `<a>` or `<button>`.

## Setup

The controller is registered on `<body>` automatically by the shell — no additional setup is required. Place `data-action="cba-view#load"` on trigger elements anywhere in the page.

```html
<button data-action="cba-view#load"
        data-view-name="detail"
        data-view-dest="#main-panel">
  Open Detail
</button>
```

## Trigger Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-view-name` | Yes | — | View name(s), pipe-separated for multi-load |
| `data-view-dest` | Yes | — | CSS selector(s) of destination element(s), pipe-separated — count must match `data-view-name` |
| `data-view-params` | No | — | JSON object of extra query params per view, pipe-separated for multi-view. Use `{}` or an empty segment for views with no params (e.g. `{"foo":1}\|{}\|{"bar":2}`) |
| `data-confirm-msg` | No | — | Shows a confirm dialog before loading |
| `data-confirm-title` | No | `"Confirm"` | Confirm dialog title |
| `data-confirm-label` | No | `"Confirm"` | Confirm button label |
| `data-cancel-label` | No | `"Cancel"` | Cancel button label |

## Controller Attribute

| Attribute | Description |
|---|---|
| `data-cba-view-url-value` | Overrides the view base URL. Default: `<meta name="cbapp-appview-url">` |

## Events

All events bubble to `document`.

| Event | Detail | Description |
|---|---|---|
| `cba-view:loaded` | `{ viewName, dest }` | View successfully loaded into destination element |
| `view:error` | `{ viewName, error }` | Fetch failed |
| `view:cancelled` | `{ trigger }` | Confirm dialog was dismissed |

## Examples

### Load a single view

```html
<button data-action="cba-view#load"
        data-view-name="detail"
        data-view-dest="#main-panel">
  Open Detail
</button>
```

### Load two views simultaneously

```html
<button data-action="cba-view#load"
        data-view-name="list|sidebar"
        data-view-dest="#main|#sidebar-panel">
  Refresh
</button>
```

### Load with extra query params

```html
<a href="#" data-action="cba-view#load"
   data-view-name="detail"
   data-view-dest="#main-panel"
   data-view-params='{"id": 42, "tab": "notes"}'>
  Open Item 42
</a>
```

### Load multiple views with per-view params

Use pipe-separated JSON objects. Use `{}` (or an empty segment) for views that need no params — positions must match `data-view-name` segments.

```html
<button data-action="cba-view#load"
        data-view-name="list|sidebar|header"
        data-view-dest="#main|#sidebar|#header"
        data-view-params='{"filter":"active"}|{}|{"prop.Name": 123}'>
  Load
</button>
```

### Load with confirm dialog

```html
<button data-action="cba-view#load"
        data-view-name="confirm-screen"
        data-view-dest="#modal"
        data-confirm-msg="Navigate away? Unsaved changes will be lost."
        data-confirm-title="Leave page?"
        data-confirm-label="Leave"
        data-cancel-label="Stay">
  Continue
</button>
```

### Listen for load events

```js
document.addEventListener("cba-view:loaded", (e) => {
  console.log("Loaded:", e.detail.viewName, e.detail.dest);
});
```

<Tip>
For `<a>` tags, `event.preventDefault()` is called automatically — no `href="#"` workaround needed, but including it is harmless.
</Tip>

## Form Submit

A `<form>` can load a view using its own fields as query params:

```html
<form data-action="submit->cba-view#submit"
      data-view-name="itemList"
      data-view-dest="#results">
  <input type="text" name="q" placeholder="Search…">
  <select name="status">
    <option value="active">Active</option>
    <option value="archived">Archived</option>
  </select>
  <button type="submit">Search</button>
</form>

<div id="results"></div>
```

Native form submission is always prevented — the form never navigates.

### Form attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-view-name` | Yes | — | View name(s), pipe-separated for multi-load |
| `data-view-dest` | Yes | — | CSS selector(s) of destination element(s) |
| `data-view-params` | No | — | JSON base params, merged **under** the form fields |
| `data-confirm-msg` | No | — | Shows a confirm dialog before loading |
| `data-confirm-title` | No | `"Confirm"` | Confirm dialog title |
| `data-confirm-label` | No | `"Confirm"` | Confirm button label |
| `data-cancel-label` | No | `"Cancel"` | Cancel button label |

Base params and fields are merged with the field winning on key conflict — the same precedence `cba-view#change` uses for a `<select>`'s value. In a multi-load, every view receives the same field set, merged over its own `data-view-params` segment.

<Tip>
Unlike `cba-view#change` — where the `<select>`'s own `name` becomes the param key — the `<form>`'s `name` attribute is never read. Each control supplies its own key, so a form needs no `name` of its own.
</Tip>

### Field limitations

Views are fetched with **GET**, so the form's fields become query string params. That imposes real constraints:

| Limitation | Behaviour |
|---|---|
| File inputs | Skipped, with a console warning. Use `cba-api` or `cba-rest` for uploads. |
| Multi-value fields | Checkbox groups and multi-selects are sent as repeated keys — `tag=a&tag=b`. |
| Unchecked checkboxes | Omitted entirely (standard `FormData` behaviour) — the server sees no key, not `false`. |
| Disabled fields | Omitted (standard `FormData` behaviour). |
| Types | Everything is stringified. The server receives `"42"` and `"true"`, never typed values. |
| Payload size | Fields ride in the URL, so long values can hit URL length limits. Use `cba-api` for large payloads. |

### Troubleshooting: the view loads but no params arrive

`FormData` only collects controls that have a `name` **and** are not disabled. Everything else is silently omitted, so the view loads cleanly with an empty query string. `cba-view#submit` logs a console warning when a form has data-carrying controls but produces no params.

| Symptom | Cause | Fix |
|---|---|---|
| View loads, no params | Controls have `id` but no `name` | Add `name="…"` — `id` is not submitted |
| View loads, params missing for some fields | Those controls are `disabled` | Use `readonly`, or re-enable before submit |
| View loads, checkbox params missing | Unchecked boxes are never submitted | Pair with a hidden input if you need an explicit value |
| Nothing happens at all | `<form>` nested inside another `<form>` | Invalid HTML — the parser drops the inner form, attributes and all. Un-nest it. |
| Nothing happens at all | `data-view-name` / `data-view-dest` on the submit button instead of the `<form>` | Move them onto the `<form>` |

Controls that sit outside the `<form>` element work if you associate them explicitly:

```html
<form id="search" data-action="submit->cba-view#submit"
      data-view-name="itemList" data-view-dest="#results"></form>

<input type="text" name="q" form="search">
```

### Conflict with cba-rest / cba-api forms

`cba-view#submit` **refuses to run** on a form that `cba-rest` or `cba-api` already owns — one carrying `data-rest-content`, `data-rest-publish`, `data-rest-workflow`, or `data-api-url`.

All three controllers listen for `submit` on `<body>`, so without the guard the REST/API operation and the view load would fire together — and `handlePostAction()` would then load the view a *second* time from the same `data-view-name` / `data-view-dest`, with different params, racing the first load on the same destination.

**What happens:** the view load is skipped and an error naming the offending attribute is logged to the console. Native form submission is still prevented, so the page does not navigate. The REST/API operation itself is unaffected and runs normally.

**What to do:** remove `data-action="submit->cba-view#submit"` from the form. On a rest/api form, `data-view-name` + `data-view-dest` already load a view once the operation succeeds — see [Shared post-action attributes](/developer-guide#shared-post-action-attributes).

```html
<!-- ✗ Guarded against — the view load is skipped and an error is logged -->
<form data-rest-content="delete"
      data-action="submit->cba-view#submit"
      data-view-name="itemList" data-view-dest="#results">
  <input type="hidden" name="contentId" value="abc-123">
  <button type="submit">Delete</button>
</form>

<!-- ✓ Correct — cba-rest runs the delete, then loads the view -->
<form data-rest-content="delete"
      data-view-name="itemList" data-view-dest="#results"
      data-view-params='{"status":"active"}'>
  <input type="hidden" name="contentId" value="abc-123">
  <button type="submit">Delete</button>
</form>
```

<Warning>
The two paths differ in what they send. `cba-view#submit` sends the **form's fields** as view params; the post-action attributes on a rest/api form send only the static `data-view-params`. If you need the submitted fields to reach the view after a REST call, read them from the `cba-rest` success event and dispatch `cbapp:load` yourself with the params you want.
</Warning>

### Combining with a refreshable dest

If the destination has `data-view-refreshable`, the submitted params are written onto it after the load — so a later `cbapp:refresh` or `data-refresh` re-runs the same search rather than resetting it.

```html
<form data-action="submit->cba-view#submit"
      data-view-name="itemList"
      data-view-dest="#results">
  <input type="text" name="q">
  <button type="submit">Search</button>
</form>

<!-- after submitting q=widgets, this element holds data-view-params='{"q":"widgets"}' -->
<div id="results" data-view-refreshable></div>
```

## Deferred Elements

A server-rendered container can declare itself **deferred** by adding `data-defer`. The controller fetches the named view and swaps it into the element itself after the page has painted — use it for regions that are slow to render server-side.

The element is its own destination: `data-view-dest` is not used, and only a single view name is supported (pipe-separated multi-load is not).

### Deferred element attributes

| Attribute | Required | Description |
|---|---|---|
| `data-defer` | Yes | Marks the element for deferred loading (presence only — no value needed) |
| `data-view-name` | Yes | Single view name to fetch into this element |
| `data-view-params` | No | JSON object of query params for the view |

### When deferred elements are discovered

- On controller connect. If the document is still parsing, discovery waits for `DOMContentLoaded` — so `cbapp-core.js` can be loaded from `<head>` without `defer`.
- After any view is injected into the page: a `cba-view#load`, a refresh, a poll, or another deferred load.

Deferred containers therefore cascade — a deferred view whose HTML contains further `data-defer` elements will load those too.

### Load lifecycle

1. The container fades out, its box size is held, and the loading indicator snaps in — the same transition as a normal load.
2. The fetched HTML is sanitized, replaces the container's contents, and fades in.
3. `data-defer` is removed **on success only**, so later discovery passes do not re-fetch it.
4. `cba-view:loaded` is dispatched with `{ viewName, dest }`.

On failure the original placeholder content snaps back, `data-defer` is **retained**, and `view:error` is dispatched. The next discovery pass retries the fetch.

Any markup left inside the container sizes the loading area, then gets replaced. An empty container reserves the remaining viewport height below it (minimum 120px) so the layout does not collapse.

<Warning>
An invalid `data-view-name` (must match `[a-zA-Z][a-zA-Z0-9_-]*`) or malformed `data-view-params` JSON is logged to the console and the fetch is skipped — no event is dispatched.
</Warning>

### Example

```html
<div id="slow-widget"
     data-defer
     data-view-name="slowReport"
     data-view-params='{"year":2026}'>
  <!-- optional placeholder; replaced once the deferred load resolves -->
</div>
```

### Combining with refreshable elements

If a deferred element also has `data-view-refreshable`, the controller writes `data-view-name` and `data-view-params` onto it after the deferred load, so `cbapp:refresh` and `data-refresh` targeting it work as usual. Adding `data-refresh-interval` starts polling as soon as the deferred load completes.

```html
<div id="job-status"
     data-defer
     data-view-refreshable
     data-view-name="jobStatus"
     data-view-params='{"jobId":"xyz"}'
     data-refresh-interval="2000"
     data-view-poll-while="[data-job-running]">
  <!-- deferred first paint, then polls while the sentinel is present -->
</div>
```

## Refreshable Elements

A container element can declare itself as a self-refreshing view by adding `data-view-refreshable`. It holds its own current view state and responds to `cbapp:refresh` events dispatched by `data-refresh` on rest/api forms after a successful operation.

### Refreshable element attributes

| Attribute | Required | Description |
|---|---|---|
| `data-view-refreshable` | Yes | Marks element as self-refreshing (presence only — no value needed) |
| `data-view-name` | Yes | Single view name — pipe-separated multi-view is not supported here |
| `data-view-params` | No | Single JSON object of query params for the current view — pipe-separated multi-view format is not supported here |

`data-view-dest` is not used — the refreshable element is always its own destination.

### How state stays current

When `cba-view#load` successfully loads into a dest element that has `data-view-refreshable`, the controller automatically updates `data-view-name` and `data-view-params` on that element. This means if different triggers load different views or params into the same dest, the element always knows its current state and `cbapp:refresh` will re-fetch the right thing.

### Example

```html
<!-- Refreshable container — declare initial view state here -->
<div id="items-list"
     data-view-refreshable
     data-view-name="itemList"
     data-view-params='{"status":"active"}'>
  <!-- populated on page load or by cba-view#load -->
</div>

<!-- Trigger that loads into the refreshable dest (updates its state automatically) -->
<a href="#" data-action="cba-view#load"
   data-view-name="itemList"
   data-view-dest="#items-list"
   data-view-params='{"status":"archived"}'>
  Show Archived
</a>

<!-- Form that triggers a refresh of #items-list after success -->
<form data-rest-content="delete"
      data-toast-success="Deleted."
      data-refresh="#items-list">
  <input type="hidden" name="contentId" value="abc-123">
  <button type="submit">Delete</button>
</form>
```

After the delete succeeds, `#items-list` re-fetches whatever view + params it currently has set — which could have been updated by a previous `cba-view#load`.
