---
title: "cbapp Developer Guide"
description: "This guide covers everything you need to know to build a ContentBase App (cbapp) on top of the cbapp-core shell. It is written for developers who are..."
---
This guide covers everything you need to know to build a ContentBase App (cbapp) on top of the `cbapp-core` shell. It is written for developers who are comfortable with HTML and basic JavaScript but may not have deep experience with front-end frameworks.

---

## Table of Contents

1. [What cbapp-core is](#what-cbapp-core-is)
2. [How the shell boots](#how-the-shell-boots)
3. [The Stimulus foundation](#the-stimulus-foundation)
4. [Meta tags reference](#meta-tags-reference)
5. [cba-view — loading view fragments](#cba-view--loading-view-fragments)
6. [cba-rest — ContentBase REST operations](#cba-rest--contentbase-rest-operations)
7. [cba-api — arbitrary API calls](#cba-api--arbitrary-api-calls)
8. [cba-ui-dropdown — dropdown menus](#cba-ui-dropdown--dropdown-menus)
9. [Modal views](#modal-views)
10. [Shared post-action attributes](#shared-post-action-attributes)
11. [Confirm dialogs](#confirm-dialogs)
12. [Toast notifications](#toast-notifications)
13. [JavaScript events reference](#javascript-events-reference)
14. [Registering your own Stimulus controllers](#registering-your-own-stimulus-controllers)
15. [Using ContentBase.callApi directly](#using-contentbasecallapi-directly)
16. [Complete worked example](#complete-worked-example)

---

## What cbapp-core is

`cbapp-core` is the **shell** that every ContentBase App runs inside. It provides:

- A standard page layout (top nav, main content area, footer)
- A JavaScript runtime that is already loaded and running before your app code does anything
- A set of **Stimulus controllers** you can use declaratively in HTML to load content, perform API calls, show toasts, and more — without writing any JavaScript yourself
- A mechanism to load your app's initial view and replace it when the user navigates

Your job as an app developer is to produce **server-rendered HTML view fragments** that the shell loads into named destination elements on the page. Most interactions are wired entirely in HTML attributes.

---

## How the shell boots

The shell template (`app.tag`) outputs a complete HTML page. The key parts are:

```html
<body data-controller="cba-view cba-rest">
  ...
  <div id="cbapp-core__main" class="cbapp-core-ui__main-container">
    <!-- your app's content is loaded here -->
  </div>
  ...
</body>
```

On `DOMContentLoaded`, the shell reads the `<meta name="cbapp-appview-url">` tag and fetches your app's initial view from that URL, injecting the result into `#cbapp-core__main` with a fade-in transition.

Any URL query parameters on the current page are forwarded to the view URL automatically. So if the user lands on `https://example.com/myapp?contentId=abc`, your view endpoint receives `?contentId=abc` appended to whatever the base URL already contains.

### What your initial view should return

Your initial view endpoint should return an HTML fragment — **not** a full HTML page. No `<html>`, `<head>`, or `<body>` tags. Just the markup you want inside `#cbapp-core__main`.

That fragment can itself contain destination elements that the user's subsequent interactions will load content into.

---

## The Stimulus foundation

cbapp-core uses [Stimulus](https://stimulus.hotwired.dev/), a lightweight JavaScript framework. You do not need to write Stimulus code yourself to use cbapp-core, but understanding one key idea helps: **controllers are connected to HTML elements via `data-controller` attributes**, and they listen for events and read other `data-*` attributes on descendant elements.

The `<body>` element has `data-controller="cba-view cba-rest"` set by the shell. This means the view and REST controllers are active on the entire page as soon as it loads. You do not wire them up yourself — they are already running.

When you need the `cba-api` or `cba-ui-dropdown` controllers, you add `data-controller="cba-api"` or `data-controller="cba-ui-dropdown"` to whichever element wraps the forms or buttons that need them.

---

## Meta tags reference

The shell reads several `<meta>` tags from `<head>`. These are set by `app.tag` and generally do not need to be changed by app developers.

| Meta name | Description |
|---|---|
| `cbapp-appview-url` | The URL the shell fetches on boot to populate `#cbapp-core__main`. Set to your app's view controller endpoint. |
| `cb-root-url` | The base URL for all REST and API calls. Resolved against paths used in `cba-rest` and `cba-api`. |
| `cb-version` | ContentBase platform version — displayed in the footer. |
| `app-name` | Your app's display name — shown in the top nav. |
| `app-version` | Your app's version string — displayed in the UI. |
| `cbapp-worker-url` | URL for the shared background web worker. |
| `cbapp-ws-url` | WebSocket endpoint for real-time task notifications. |

---

## cba-view — loading view fragments

`cba-view` is mounted on `<body>` by the shell. It is responsible for loading HTML fragments from the server and injecting them into destination elements on the page.

### Basic usage

Add `data-action="cba-view#load"` to any link or button, along with a view name and destination selector:

```html
<a href="#"
   data-action="cba-view#load"
   data-view-name="itemDetail"
   data-view-dest="#detail-panel">
  Open detail
</a>

<div id="detail-panel" class="flex-1 min-h-48">
  <!-- itemDetail view loads here -->
</div>
```

When the user clicks the link, the controller fetches the `itemDetail` view from the server, fades out the current contents of `#detail-panel`, shows a loading indicator, then fades in the new content when it arrives.

The `href="#"` on the anchor is ignored — `cba-view` prevents default navigation on `<a>` elements automatically.

### How the view URL is built

The view controller reads `<meta name="cbapp-appview-url">` and appends `?_view=<viewName>` to it. So if the meta content is `/publisher/toolView.ajx?tool=myapp` and the view name is `itemDetail`, the fetch URL is:

```
/publisher/toolView.ajx?tool=myapp&_view=itemDetail
```

Your server receives the `_view` parameter and renders the appropriate fragment.

### Passing parameters to a view

Use `data-view-params` with a JSON object:

```html
<button data-action="cba-view#load"
        data-view-name="itemDetail"
        data-view-dest="#detail-panel"
        data-view-params='{"contentId":"abc-123","tab":"notes"}'>
  Open notes
</button>
```

These are appended to the fetch URL as query parameters: `&contentId=abc-123&tab=notes`.

### Loading multiple views at once

Use pipe-separated (`|`) values. Each view name maps positionally to the corresponding destination:

```html
<button data-action="cba-view#load"
        data-view-name="itemList|detailPanel"
        data-view-dest="#list|#detail">
  Refresh both
</button>
```

Parameters also use pipe separators. Use an empty segment for views with no params:

```html
data-view-params='{"status":"active"}||{"id":"99"}'
<!-- viewA gets status=active, viewB gets nothing, viewC gets id=99 -->
```

### Loading a view from a form submit

Put `data-action="submit->cba-view#submit"` on a `<form>` and its fields are collected and sent as the view's query params. Native submission is prevented — the page never navigates.

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

Submitting with `q=widgets` and `status=active` fetches:

```
/publisher/toolView.ajx?tool=myapp&_view=itemList&q=widgets&status=active
```

`data-view-params` still works as a base — form fields are merged over it, so a field wins on key conflict:

```html
<form data-action="submit->cba-view#submit"
      data-view-name="itemList"
      data-view-dest="#results"
      data-view-params='{"repo":"main","limit":"50"}'>
  <input type="text" name="q">
  <button type="submit">Search</button>
</form>
```

Pipe-separated multi-load works too — every view receives the same field set, merged over its own params segment.

If the destination is `data-view-refreshable`, the submitted params are written onto it after the load, so a later `cbapp:refresh` re-runs the same search instead of resetting it.

> Unlike `cba-view#change` — where the `<select>`'s own `name` becomes the param key — the `<form>`'s `name` attribute is never read. Each control supplies its own key, so the form itself needs no `name`.

#### Field limitations

Views are fetched with **GET**, so fields become query string params. This is deliberate — it keeps searches and filters bookmarkable and re-runnable — but it constrains what a form can carry:

- **File inputs are skipped** (with a console warning). Use `cba-api` or `cba-rest` for uploads.
- **Multi-value fields** — checkbox groups and multi-selects — are sent as repeated keys: `tag=a&tag=b`.
- **Unchecked checkboxes and disabled fields are omitted entirely**, per standard `FormData` behaviour. The server sees no key at all, not `false`.
- **Everything is stringified.** The server receives `"42"`, never a number.
- **Keep payloads small.** Fields ride in the URL, so long values can hit URL length limits.

#### Troubleshooting: the view loads but no params arrive

`FormData` only collects controls that have a `name` **and** are not disabled — everything else is silently omitted, so the view loads with an empty query string and no error. `cba-view#submit` logs a console warning when a form has data-carrying controls but yields no params.

- **Controls have `id` but no `name`** — `id` is not submitted. Add `name`.
- **Controls are `disabled`** — omitted entirely. Use `readonly` instead, or re-enable before submit.
- **Unchecked checkboxes** — never submitted. Pair with a hidden input if the server needs an explicit value.
- **Nothing happens at all** — either the `<form>` is nested inside another `<form>` (invalid HTML; the parser drops the inner one, attributes and all), or `data-view-name`/`data-view-dest` are on the submit button instead of the `<form>`.

Controls outside the `<form>` work if you associate them explicitly with `form="<form-id>"`.

#### Conflict with cba-rest / cba-api forms

`cba-view#submit` **refuses to run** on a form that `cba-rest` or `cba-api` already owns — one carrying `data-rest-content`, `data-rest-publish`, `data-rest-workflow`, or `data-api-url`.

All three controllers listen for `submit` on `<body>`. Without the guard the REST/API operation and the view load would fire together, and the post-action handler would then load the view a *second* time from the same `data-view-name` / `data-view-dest` — two loads racing on one destination with different params.

- **What happens:** the view load is skipped and an error naming the offending attribute is logged to the console. Native submission is still prevented, so the page does not navigate. The REST/API operation runs normally.
- **What to do:** remove `data-action="submit->cba-view#submit"`. On a rest/api form, `data-view-name` + `data-view-dest` already load a view after the operation succeeds — see [Shared post-action attributes](#shared-post-action-attributes).

```html
<!-- Guarded against: view load skipped, error logged -->
<form data-rest-content="delete"
      data-action="submit->cba-view#submit"
      data-view-name="itemList" data-view-dest="#results">
  …
</form>

<!-- Correct: cba-rest runs the delete, then loads the view -->
<form data-rest-content="delete"
      data-view-name="itemList" data-view-dest="#results"
      data-view-params='{"status":"active"}'>
  …
</form>
```

The two paths send different things: `cba-view#submit` sends the **form's fields** as view params, while the post-action attributes on a rest/api form send only the static `data-view-params`. If you need submitted fields to reach a view after a REST call, listen for the `cba-rest` success event and dispatch `cbapp:load` yourself with the params you want.

### Refreshable elements

A destination element can declare itself **refreshable** — meaning it remembers which view it contains and can reload itself when asked.

```html
<div id="items-list"
     data-view-refreshable
     data-view-name="itemList"
     data-view-params='{"status":"active"}'>
  <!-- server-rendered initial content here -->
</div>
```

When `cba-view#load` successfully loads a view into a `data-view-refreshable` element, the element's `data-view-name` and `data-view-params` attributes are updated automatically to reflect the new view state.

To trigger a refresh programmatically (or from a REST/API action), dispatch a `cbapp:refresh` event on the element:

```javascript
document.getElementById("items-list")
  .dispatchEvent(new CustomEvent("cbapp:refresh", { bubbles: true }));
```

More commonly you don't do this yourself — you use `data-refresh` on a form or button and the framework dispatches it for you (see [Shared post-action attributes](#shared-post-action-attributes)).

### Polling

A refreshable element can poll the server on an interval while a condition is true:

```html
<div id="job-status"
     data-view-refreshable
     data-view-name="jobStatus"
     data-view-params='{"jobId":"xyz"}'
     data-refresh-interval="2000"
     data-view-poll-while="[data-job-running]">
  <!-- server renders <span data-job-running> while the job is in progress -->
</div>
```

- `data-refresh-interval` — polling interval in milliseconds
- `data-view-poll-while` — CSS selector; polling continues while a matching child element is present. Defaults to `[data-job-running]` if omitted.

Polling starts automatically when the element is first populated (via load or refresh) and the sentinel element is found. It stops silently when the server no longer renders the sentinel. Poll refreshes are silent — no fade transition — to avoid distracting the user during background updates.

### Deferred loading

A server-rendered container can fetch its own content *after* the page has painted by adding `data-defer`. Use it for regions that are slow to render server-side — the shell paints the page immediately and fills the deferred region in the background.

```html
<div id="slow-widget"
     data-defer
     data-view-name="slowReport"
     data-view-params='{"year":2026}'>
  <!-- optional placeholder markup -->
</div>
```

The element is its **own destination** — `data-view-dest` is not used, and only a single view name is supported (no pipe-separated multi-load).

**When deferred elements are discovered**

- When the view controller connects. If the document is still parsing, discovery waits for `DOMContentLoaded`, so `cbapp-core.js` can be loaded from `<head>` without `defer`.
- After *any* view is injected into the page — a `cba-view#load`, a refresh, a poll, or another deferred load.

Deferred elements therefore cascade: a deferred view whose HTML contains further `data-defer` containers will load those too.

**Lifecycle of a deferred load**

1. The container fades out, its box size is held, and the loading indicator snaps in — the same transition as a normal `cba-view#load`.
2. The view is fetched; the sanitized HTML replaces the container's contents and fades in.
3. `data-defer` is **removed on success**, so later discovery passes don't re-fetch it.
4. `cba-view:loaded` fires with `{ viewName, dest }`.

On failure the original placeholder content snaps back, `data-defer` is **kept**, and `view:error` fires — the next discovery pass (the next time any view loads into the page) retries the fetch.

Any markup you leave inside the container is used to size the loading area, then replaced. If the container is empty, the shell reserves the remaining viewport height below it (minimum 120px) so the layout doesn't collapse.

Misconfiguration is logged to the console and the fetch is skipped — a `data-view-name` that isn't a valid view name (`[a-zA-Z][a-zA-Z0-9_-]*`), or a `data-view-params` value that isn't valid JSON.

**Combining with refreshable and polling**

Add `data-view-refreshable` to make a deferred container self-refreshing once loaded. The controller writes `data-view-name` and `data-view-params` onto the element after the deferred fetch, so `cbapp:refresh` and `data-refresh` targeting it work as usual. Add `data-refresh-interval` and polling starts as soon as the deferred load completes:

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

### Loading a view programmatically via event

Any element can trigger a view load by dispatching a `cbapp:load` event on itself:

```javascript
document.getElementById("my-dest").dispatchEvent(
  new CustomEvent("cbapp:load", {
    bubbles: true,
    cancelable: false,
    detail: { viewName: "someView" }
  })
);
```

When the destination is the modal content element (`#cbapp__modal`), include modal configuration in the detail:

```javascript
document.getElementById("cbapp__modal").dispatchEvent(
  new CustomEvent("cbapp:load", {
    bubbles: true,
    cancelable: false,
    detail: {
      viewName: "editForm",
      modalSize: "lg",
      modalTitle: "Edit Item",
      modalNoDismiss: false
    }
  })
);
```

| `detail` field | Default | Description |
|---|---|---|
| `viewName` | required | View name to fetch |
| `modalSize` | `"md"` | Panel width: `"sm"` \| `"md"` \| `"lg"` |
| `modalTitle` | `""` | Header title; hidden when empty |
| `modalNoDismiss` | `false` | Set to `true` to disable Escape / click-outside close |

This is the mechanism used by `data-load-view` / `data-load-dest` post-action attributes (see below).

### Clearing a destination with `showBlank`

`showBlank` is a reserved view name that clears a destination element without making a network request. No base URL is required.

```html
<!-- Clears #detail-panel when clicked -->
<button data-action="cba-view#load"
        data-view-name="showBlank"
        data-view-dest="#detail-panel">
  Clear panel
</button>

<!-- Works in multi-load — clears one dest, fetches the other -->
<button data-action="cba-view#load"
        data-view-name="showBlank|itemList"
        data-view-dest="#detail-panel|#list-panel">
  Back to list
</button>
```

The standard fade-out / fade-in transition still runs. If the destination is a `data-view-refreshable` element, its `data-view-name` and `data-view-params` attributes are also cleared.

`showBlank` also works via the `cbapp:load` event:

```javascript
document.getElementById("detail-panel").dispatchEvent(
  new CustomEvent("cbapp:load", {
    bubbles: true,
    cancelable: false,
    detail: { viewName: "showBlank" }
  })
);
```

### Loading transitions

When `cba-view#load` is triggered:

1. The destination element **fades out** immediately (150ms). The fetch runs in parallel.
2. A **loading indicator** snaps into the (now-invisible) element.
3. When the fetch returns successfully, the new content **snaps in** at opacity 0 and **fades in** (150ms).
4. On fetch error, the **original content snaps back** with no transition — intentionally abrupt to signal a problem.

Rapid re-clicks are safe: if the user clicks again before the first fetch completes, the original content (not the loading indicator) is held as the backup.

### Destination element CSS

For the loading indicator to centre correctly:

- The destination should have a defined height, either from its content, from `min-h-*`, or from being a `flex-1` child of a container with a defined height.
- The `.cbapp-core-ui__load` class handles centering — you should not need to add anything to the dest itself.
- If the dest starts **empty**, add `min-h-*` to give it a baseline size:

  ```html
  <div id="detail-panel" class="flex-1 min-h-48">...</div>
  ```

### cba-view attribute summary

**On the trigger element** (`<a>` or `<button>` with `data-action="cba-view#load"`):

| Attribute | Required | Description |
|---|---|---|
| `data-view-name` | Yes | View name, or pipe-separated names for multi-load |
| `data-view-dest` | Yes | CSS selector for the destination, or pipe-separated selectors |
| `data-view-params` | No | JSON query params, or pipe-separated JSON segments |
| `data-confirm-msg` | No | Show a confirm dialog before loading |
| `data-confirm-title` | No | Confirm dialog title (default: "Confirm") |
| `data-confirm-label` | No | Confirm button label (default: "Confirm") |
| `data-cancel-label` | No | Cancel button label (default: "Cancel") |
| `data-modal-size` | No | When dest is `#cbapp__modal`: panel width — `"sm"`, `"md"` (default), or `"lg"` |
| `data-modal-title` | No | When dest is `#cbapp__modal`: text shown in the modal header |
| `data-modal-no-dismiss` | No | When dest is `#cbapp__modal`: presence disables Escape / click-outside close |

**On a form element** (`data-action="submit->cba-view#submit"`):

| Attribute | Required | Description |
|---|---|---|
| `data-view-name` | Yes | View name, or pipe-separated names for multi-load |
| `data-view-dest` | Yes | CSS selector for the destination, or pipe-separated selectors |
| `data-view-params` | No | JSON base params — form fields are merged over these |
| `data-confirm-msg` | No | Show a confirm dialog before loading |

**On a deferred container element**:

| Attribute | Required | Description |
|---|---|---|
| `data-defer` | Yes | Marks the element for deferred loading (presence only, no value) |
| `data-view-name` | Yes | Single view name to fetch into this element — pipe-separated multi-load is not supported |
| `data-view-params` | No | JSON query params for the view |

**On a refreshable destination element**:

| Attribute | Required | Description |
|---|---|---|
| `data-view-refreshable` | Yes | Marks element as refreshable (presence only, no value) |
| `data-view-name` | Yes | Current view loaded into this element |
| `data-view-params` | No | Current params as a JSON object |
| `data-refresh-interval` | No | Polling interval in ms |
| `data-view-poll-while` | No | CSS selector for the polling sentinel (default: `[data-job-running]`) |

---

## cba-rest — ContentBase REST operations

`cba-rest` is mounted on `<body>` by the shell. It intercepts form submissions and button clicks and maps them to ContentBase content and publish API operations.

You do not add `data-controller="cba-rest"` yourself — it is already active on the entire page.

### How it works

Add a `data-rest-{category}="operation"` attribute to a `<form>` or `<button>`. On submit (form) or click (button), the controller:

1. Optionally shows a confirm dialog
2. Collects the fields
3. Makes the API call
4. Runs post-action steps: shows a toast, triggers a view refresh, etc.

### Supported categories and operations

#### `data-rest-content` — content lifecycle operations

| Operation | HTTP | Path | Description |
|---|---|---|---|
| `load` | POST/PATCH | `webresources/lcm/repo/xmsId[/{contentId}]` | Create or update content (multipart) |
| `setProperties` | PATCH | `webresources/lcm/repo/xmsId/{contentId}/properties` | Update metadata fields |
| `delete` | PATCH | `webresources/lcm/repo/{contentId}/delete` | Soft-delete content |
| `restore` | PATCH | `webresources/lcm/repo/{contentId}/restore` | Restore soft-deleted content |
| `pass` | POST/PATCH | `webresources/lcm/repo/{contentId}/pass[/{tagName}]` | Pass content to a workflow step |

#### `data-rest-publish` — publish queue operations

| Operation | HTTP | Path | Description |
|---|---|---|---|
| `queue` | POST | `webresources/pub/publication/{pubname}/queue` | Queue content for publication |

### Form usage

Form fields become the request body. Use hidden inputs to pass required parameters like `contentId`.

```html
<form data-rest-content="setProperties"
      data-toast-success="Saved."
      data-toast-error="Save failed."
      data-refresh="#items-list">

  <input type="hidden" name="contentId" value="abc-123">
  <input type="text" name="title" value="My Item">
  <input type="text" name="description" value="A description">

  <button type="submit">Save</button>
</form>
```

### Button usage (no form needed)

Pass parameters as JSON in `data-rest-params`:

```html
<button data-rest-content="delete"
        data-rest-params='{"contentId":"abc-123"}'
        data-confirm-msg="Delete this item? This cannot be undone."
        data-toast-success="Deleted."
        data-refresh="#items-list">
  Delete
</button>
```

### File uploads

For file upload forms (using the `load` operation), add `<input type="file">` to your form. The controller detects file inputs automatically and sends the request as multipart form data.

To also capture the file's last-modified timestamp, add `data-file-modified="<fieldName>"` to the file input:

```html
<input type="file" name="file" data-file-modified="modified">
```

This injects an extra field `modified` containing the file's `lastModified` timestamp (milliseconds since epoch).

### Publish queue example

```html
<button data-rest-publish="queue"
        data-rest-params='{"pubname":"my-publication","contentId":"abc-123"}'
        data-confirm-msg="Queue this item for publication?"
        data-toast-success="Queued for publication.">
  Publish
</button>
```

### cba-rest attribute summary

**On form or button**:

| Attribute | Required | Description |
|---|---|---|
| `data-rest-content` | * | Operation name: `load`, `setProperties`, `delete`, `restore`, `pass` |
| `data-rest-publish` | * | Operation name: `queue` |
| `data-rest-params` | No | JSON params object (buttons only — forms use field inputs) |
| `data-toast-success` | No | Toast message on success |
| `data-toast-error` | No | Toast message on error (falls back to server message or error message) |
| `data-refresh` | No | CSS selector of a refreshable element to reload on success |
| `data-load-view` | No | View name to load into `data-load-dest` on success |
| `data-load-dest` | No | CSS selector of the dest to load into (required when `data-load-view` is set) |
| `data-modal-close` | No | Close the modal after a successful operation (presence only) |
| `data-modal-size` | No | When `data-load-dest="#cbapp__modal"`: panel width for the loaded view |
| `data-modal-title` | No | When `data-load-dest="#cbapp__modal"`: header title for the loaded view |
| `data-confirm-msg` | No | Confirmation question shown in a dialog before the operation runs |
| `data-confirm-title` | No | Confirm dialog title (default: "Confirm") |
| `data-confirm-label` | No | Confirm button label (default: "Confirm") |
| `data-cancel-label` | No | Cancel button label (default: "Cancel") |

*One of these is required.

---

## cba-api — arbitrary API calls

`cba-api` works like `cba-rest` but for calls to arbitrary endpoints rather than the standard ContentBase operation paths. You add `data-controller="cba-api"` to a wrapper element yourself.

```html
<div data-controller="cba-api">
  <form data-api-url="/api/my-endpoint" data-api-method="PATCH"
        data-toast-success="Updated." data-refresh="#my-view">
    <input name="value" value="...">
    <button type="submit">Update</button>
  </form>
</div>
```

The URL in `data-api-url` is resolved against `<meta name="cb-root-url">`. Absolute URLs (`http://...`) are used as-is.

### Form usage

All form fields are collected and sent as the request body (FormData). File inputs work the same as with `cba-rest`, including `data-file-modified`.

```html
<div data-controller="cba-api">
  <form data-api-url="/api/items"
        data-api-method="POST"
        data-toast-success="Item created."
        data-load-view="itemDetail"
        data-load-dest="#detail-panel">
    <input name="title">
    <button type="submit">Create</button>
  </form>
</div>
```

### Button usage (no form needed)

```html
<div data-controller="cba-api">
  <button data-api-url="/api/items/abc-123"
          data-api-method="DELETE"
          data-api-params='{"reason":"outdated"}'
          data-confirm-msg="Permanently delete?"
          data-toast-success="Deleted."
          data-refresh="#items-list">
    Delete
  </button>
</div>
```

### cba-api attribute summary

**On form**:

| Attribute | Required | Description |
|---|---|---|
| `data-api-url` | Yes | Endpoint path or absolute URL |
| `data-api-method` | No | HTTP method (default: `POST`) |
| `data-toast-success` | No | Toast on success |
| `data-toast-error` | No | Toast on error |
| `data-refresh` | No | Selector of refreshable element to reload on success |
| `data-load-view` | No | View name to load on success |
| `data-load-dest` | No | Dest selector for `data-load-view` |
| `data-confirm-msg` | No | Confirm dialog message |
| `data-confirm-title` | No | Confirm dialog title |
| `data-confirm-label` | No | Confirm button label |
| `data-cancel-label` | No | Cancel button label |

**On button** (all form attributes, plus):

| Attribute | Required | Description |
|---|---|---|
| `data-api-params` | No | JSON object sent as the request body |

---

## cba-ui-dropdown — dropdown menus

`cba-ui-dropdown` handles open/close behaviour for dropdown menus. Add `data-controller="cba-ui-dropdown"` to the wrapper and `data-cba-ui-dropdown-target="menu"` to the panel that should show and hide.

```html
<div data-controller="cba-ui-dropdown" class="relative">

  <button data-action="cba-ui-dropdown#toggle">
    Options ▾
  </button>

  <div data-cba-ui-dropdown-target="menu" class="hidden absolute ...">
    <a href="#">Edit</a>
    <a href="#">Delete</a>
  </div>

</div>
```

The menu starts hidden (`class="hidden"`). Clicking the toggle button shows it. The dropdown closes automatically when:
- The user clicks anywhere outside the dropdown
- The user presses `Escape`

You can also wire separate open and close actions if needed:

```html
<button data-action="cba-ui-dropdown#open">Open</button>
<button data-action="cba-ui-dropdown#close">Close</button>
```

---

## Modal views

The shell includes a built-in modal panel. When you set `data-view-dest="#cbapp__modal"` on any `cba-view#load` trigger, the view is loaded into the modal instead of an inline element.

The modal opens immediately on click (showing a loading indicator), fades in the content when the fetch completes, and closes if the fetch fails. One modal is active at a time — triggering a new modal while one is open replaces the content.

### Basic usage

```html
<button data-action="cba-view#load"
        data-view-name="editForm"
        data-view-dest="#cbapp__modal"
        data-modal-size="md"
        data-modal-title="Edit Record">
  Edit
</button>
```

### Sizing

Three named widths are available via `data-modal-size`. The height is always determined by the loaded content, growing up to 90% of the viewport height before the panel starts scrolling.

| Value | Max width | When to use |
|---|---|---|
| `sm` | 384px | Simple forms, short confirmations with custom content |
| `md` | 576px | Default — most CRUD forms and detail views |
| `lg` | 768px | Complex forms, multi-column layouts |

If `data-modal-size` is omitted, `"md"` is used.

### Title

`data-modal-title` sets the text displayed in the modal header row. Omit it to hide the header title entirely (the close button remains).

```html
data-modal-title="Upload File"
```

### Closing the modal from inside the loaded view

Add `data-modal-close` to any non-submit button inside the loaded view to close the modal immediately when clicked:

```html
<button type="button" data-modal-close>Cancel</button>
```

The modal also closes when:
- The user clicks the **×** button in the header
- The user presses **Escape**
- The user clicks **outside** the modal panel (on the backdrop)

### Disabling click-outside / Escape dismiss

Add `data-modal-no-dismiss` to the trigger element to prevent the user from closing the modal by clicking outside it or pressing Escape. The × button and `data-modal-close` buttons inside the view still work.

```html
<button data-action="cba-view#load"
        data-view-name="importWizard"
        data-view-dest="#cbapp__modal"
        data-modal-title="Import"
        data-modal-no-dismiss>
  Import
</button>
```

Use this for flows where accidental dismissal could lose meaningful user progress.

### Closing the modal after a REST action

Add `data-modal-close` to the **form element** (not the submit button) to close the modal automatically when the REST operation succeeds:

```html
<form data-rest-content="setProperties"
      data-toast-success="Saved."
      data-refresh="#items-list"
      data-modal-close>
  <input type="hidden" name="contentId" value="abc-123">
  <input type="text" name="title" value="My Item">
  <button type="submit">Save</button>
  <button type="button" data-modal-close>Cancel</button>
</form>
```

Here the Cancel button closes the modal immediately (no action taken), while the form's `data-modal-close` closes it only after a successful save. This is the recommended pattern for modal edit forms.

`data-modal-close` works on `cba-api` forms and buttons the same way.

### Loading a different view into the already-open modal

Use `data-load-dest="#cbapp__modal"` and `data-load-view` together (the standard post-action pair) when a REST or API operation should navigate to a new view inside the same modal. Include `data-modal-size` and `data-modal-title` on the source form or button if you need to update the panel configuration.

```html
<!-- Step 1 form — after create, advance to the edit form in the same modal -->
<form data-rest-content="load"
      data-toast-success="Created."
      data-load-dest="#cbapp__modal"
      data-load-view="editForm"
      data-modal-title="Edit New Item">
  <input name="title">
  <button type="submit">Create</button>
</form>
```

### Modal attribute summary

**On the `cba-view#load` trigger element** (in addition to standard view attributes):

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-modal-size` | No | `"md"` | Panel width: `"sm"` \| `"md"` \| `"lg"` |
| `data-modal-title` | No | `""` | Text in the modal header; header title hidden when empty |
| `data-modal-no-dismiss` | No | — | Presence disables Escape and click-outside close |

**Inside the loaded view**:

| Attribute | Where | Description |
|---|---|---|
| `data-modal-close` | `<button type="button">` | Close the modal immediately on click |
| `data-modal-close` | `<form>` | Close the modal on successful REST / API action |

---

## Shared post-action attributes

The following attributes work on any form or button handled by `cba-rest` or `cba-api`. They run after the operation completes.

### `data-toast-success`

Shows a success toast notification when the operation succeeds.

```html
data-toast-success="Item saved successfully."
```

If omitted, no toast is shown on success.

### `data-toast-error`

Shows a toast when the operation fails. If omitted, the framework shows the server's error message if available, or the raw error message.

```html
data-toast-error="Could not save. Please try again."
```

### `data-refresh`

On success, dispatches a `cbapp:refresh` event to the matching `data-view-refreshable` element, causing it to reload its view from the server.

```html
data-refresh="#items-list"
```

The selector must match a `data-view-refreshable` element. If the selector doesn't match anything in the DOM, a warning is logged to the console.

### `data-load-view` + `data-load-dest`

On success, loads a named view into a destination element. Both attributes must be present together.

```html
data-load-view="itemDetail"
data-load-dest="#detail-panel"
```

This dispatches a `cbapp:load` event on the dest element, which the view controller handles — including the fade-out/loading/fade-in transition.

When `data-load-dest="#cbapp__modal"`, also include `data-modal-size` and `data-modal-title` on the form or button if you want to update the panel configuration when the new view loads.

### `data-modal-close`

On a form or button handled by `cba-rest` or `cba-api`, closes the modal after the operation succeeds.

```html
data-modal-close
```

This is a presence attribute — no value needed. It has no effect when the modal is not open.

---

## Confirm dialogs

`data-confirm-msg` works on any interactive element in the app:

- **`cba-rest` and `cba-api` forms and buttons** — dialog shows before the operation fires
- **`cba-view#load` triggers** — dialog shows before the view fetch begins
- **Plain `<a href>` links** — dialog shows before the browser navigates; works with any link in your view, no controller wiring needed

```html
<!-- REST button -->
<button data-rest-content="delete"
        data-rest-params='{"contentId":"abc"}'
        data-confirm-msg="Are you sure you want to delete this item?"
        data-confirm-title="Delete item"
        data-confirm-label="Delete"
        data-cancel-label="Keep it">
  Delete
</button>

<!-- View trigger -->
<a data-action="cba-view#load"
   data-view-name="newScreen"
   data-view-dest="#main"
   data-confirm-msg="Navigate away? Unsaved changes will be lost.">
  Go somewhere else
</a>

<!-- Plain link — confirms before navigating, no controller needed -->
<a href="/reports/heavy-export"
   data-confirm-msg="This report can take a minute or two to generate. Continue?"
   data-confirm-title="Generate report"
   data-confirm-label="Continue">
  Download full export
</a>
```

The dialog is a modal with a backdrop. It closes on cancel, confirm, or pressing `Escape`. If the user cancels, the operation does not run. For plain links, navigation is simply suppressed; for controller-wired elements a `*:cancelled` event is dispatched.

Plain links that open in a new tab (`target="_blank"`) are also handled — the new tab opens via `window.open` with `noopener,noreferrer` after confirmation.

**Confirm dialog attributes** (available on all element types above):

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-confirm-msg` | Yes | — | Message shown in the dialog body |
| `data-confirm-title` | No | `"Confirm"` | Dialog heading |
| `data-confirm-label` | No | `"Confirm"` | Confirm button label |
| `data-cancel-label` | No | `"Cancel"` | Cancel button label (not available on plain links) |

---

## Toast notifications

Toasts appear at the top-centre of the viewport and auto-dismiss after 4 seconds. There are two completely independent ways to trigger them, and they do not interfere with each other.

### Standalone toasts — `data-toast`

Add `data-toast` to any element to show a toast immediately when it is clicked. No controller wiring is needed. On `<a>` elements, navigation is prevented.

This is the recommended approach for placeholder links and "not yet implemented" features.

```html
<!-- Link placeholder — shows a warning toast, does not navigate -->
<a href="#" data-toast="Not yet implemented">Reports</a>

<!-- Explicit variant -->
<a href="#" data-toast="Coming in the next release" data-toast-variant="info">Analytics</a>

<!-- Works on buttons too -->
<button type="button" data-toast="This feature is coming soon" data-toast-variant="info">
  Export CSV
</button>
```

**Attributes:**

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-toast` | Yes | — | Message to display in the toast |
| `data-toast-variant` | No | `"warning"` | `"info"` \| `"success"` \| `"warning"` \| `"error"` |

### Post-action toasts — `data-toast-success` / `data-toast-error`

These are **not** click handlers. They are read by the shell after a `cba-rest` or `cba-api` operation completes, and they only make sense on forms and buttons connected to those controllers. See [Shared post-action attributes](#shared-post-action-attributes) for full details.

```html
<!-- Only fires after the REST delete operation succeeds -->
<button data-rest-content="delete"
        data-rest-params='{"contentId":"abc"}'
        data-toast-success="Item deleted."
        data-toast-error="Could not delete. Please try again.">
  Delete
</button>
```

Putting `data-toast-success` or `data-toast-error` on a plain link or a button outside a `cba-rest`/`cba-api` context has no effect — those attributes are only read by `handlePostAction`, not by any click listener.

---

## JavaScript events reference

All controllers dispatch custom events that bubble to `document`. You can listen for them anywhere to add custom behaviour.

### cba-view events

| Event | When | `event.detail` |
|---|---|---|
| `cba-view:loaded` | View successfully injected | `{ viewName, dest }` |
| `view:error` | Fetch failed | `{ viewName, error }` |
| `view:cancelled` | Confirm dialog dismissed | `{ trigger }` |

### cba-rest events

| Event | When | `event.detail` |
|---|---|---|
| `cba-rest:success` | Operation succeeded | `{ trigger, operation, result }` |
| `cba-rest:error` | Operation failed | `{ trigger, operation, error }` |
| `rest:cancelled` | Confirm dialog dismissed | `{ trigger, operation }` |

### cba-api events

| Event | When | `event.detail` |
|---|---|---|
| `cba-api:success` | Request succeeded | `{ form, result }` |
| `cba-api:error` | Request failed | `{ form, error }` |
| `cba-api:cancelled` | Confirm dialog dismissed | `{ form }` |

Note: for button triggers on `cba-api`, `form` in the event detail is the button element.

### Listening example

```javascript
document.addEventListener("cba-rest:success", (event) => {
  const { operation, result } = event.detail;
  if (operation === "delete") {
    console.log("Deleted successfully", result);
  }
});

document.addEventListener("cba-view:loaded", (event) => {
  const { viewName, dest } = event.detail;
  // Run setup code after a specific view loads
  if (viewName === "itemDetail") {
    initSomethingIn(dest);
  }
});
```

---

## Registering your own Stimulus controllers

The shell exposes a `ContentBase` global object. Use `ContentBase.register()` to register your own Stimulus controllers:

```javascript
// In your app's JS file (loaded after cbapp-core.js)
ContentBase.register("my-app-widget", MyWidgetController);
```

Your controller class should extend `ContentBase.Controller`, which is the Stimulus `Controller` base class:

```javascript
class MyWidgetController extends ContentBase.Controller {
  connect() {
    console.log("widget connected");
  }

  doSomething() {
    // ...
  }
}

ContentBase.register("my-app-widget", MyWidgetController);
```

Then use it in your view fragments:

```html
<div data-controller="my-app-widget">
  <button data-action="my-app-widget#doSomething">Do it</button>
</div>
```

Stimulus registers all controllers on the `<body>` element's Stimulus application instance, so your controller works anywhere in the page including content loaded dynamically by `cba-view`.

---

## Using ContentBase.callApi directly

If you need to make an API call from your own JavaScript (not via a form or button), use `ContentBase.callApi`:

```javascript
ContentBase.callApi("/api/my-endpoint", {
  method: "POST",
  body: { contentId: "abc-123", title: "New title" },
})
  .then((result) => {
    console.log("Success", result);
  })
  .catch((err) => {
    console.error("Failed", err);
  });
```

The path is resolved against `<meta name="cb-root-url">`. Absolute URLs are used as-is. Standard headers (`Accept`, `X-Requested-With`, `credentials: same-origin`) are added automatically. 401 responses redirect to the login page automatically.

The body can be:
- A plain object `{ key: value }` — sent as form-encoded data
- A `FormData` instance — sent as-is
- A `FormData` instance containing `File` objects — sent as multipart

`callApi` returns a Promise that resolves with the parsed JSON response body, or `{}` for non-JSON responses.

---

## Complete worked example

This example shows a typical app pattern: a list on the left, a detail panel on the right, with operations on items.

### Initial view fragment (returned by your view endpoint at `_view=` unset or `_view=appShell`)

```html
<div class="flex gap-4 h-full">

  <!-- Left: item list -->
  <div id="items-list"
       class="w-64 flex-none"
       data-view-refreshable
       data-view-name="itemList"
       data-view-params='{"status":"active"}'>
    <!-- server renders the list here on first load -->
    <p>Loading...</p>
  </div>

  <!-- Right: detail panel (empty until user selects an item) -->
  <div id="detail-panel" class="flex-1 min-h-48">
    <p class="text-gray-400 text-sm">Select an item to view details.</p>
  </div>

</div>
```

### Item list fragment (returned by your view endpoint at `_view=itemList`)

```html
<ul>
  <li>
    <a data-action="cba-view#load"
       data-view-name="itemDetail"
       data-view-dest="#detail-panel"
       data-view-params='{"contentId":"abc-123"}'>
      My Item
    </a>
  </li>
  <li>
    <a data-action="cba-view#load"
       data-view-name="itemDetail"
       data-view-dest="#detail-panel"
       data-view-params='{"contentId":"def-456"}'>
      Another Item
    </a>
  </li>
</ul>
```

### Item detail fragment (returned by your view endpoint at `_view=itemDetail&contentId=abc-123`)

```html
<div>
  <h2>My Item</h2>

  <!-- Edit form -->
  <form data-rest-content="setProperties"
        data-toast-success="Saved."
        data-toast-error="Save failed."
        data-refresh="#items-list">
    <input type="hidden" name="contentId" value="abc-123">
    <input type="text" name="title" value="My Item">
    <button type="submit">Save</button>
  </form>

  <!-- Delete button -->
  <button data-rest-content="delete"
          data-rest-params='{"contentId":"abc-123"}'
          data-confirm-msg="Delete this item? This cannot be undone."
          data-confirm-title="Delete item"
          data-confirm-label="Yes, delete"
          data-cancel-label="Cancel"
          data-toast-success="Item deleted."
          data-refresh="#items-list"
          data-load-view="emptyDetail"
          data-load-dest="#detail-panel">
    Delete
  </button>

  <!-- Publish button -->
  <button data-rest-publish="queue"
          data-rest-params='{"pubname":"my-pub","contentId":"abc-123"}'
          data-confirm-msg="Queue this item for publication?"
          data-toast-success="Queued for publication.">
    Publish
  </button>
</div>
```

### What happens when the user clicks Delete

1. The confirm dialog appears: "Delete this item? This cannot be undone."
2. The user clicks "Yes, delete."
3. `cba-rest` sends `PATCH /webresources/lcm/repo/abc-123/delete`.
4. On success:
   - A success toast "Item deleted." appears.
   - `#items-list` receives a `cbapp:refresh` event and reloads its `itemList` view (the deleted item disappears from the list).
   - `#detail-panel` receives a `cbapp:load` event for `emptyDetail`, fading in a "nothing selected" state.

All of this is wired entirely in HTML attributes — no JavaScript written.

---

## Tips and common mistakes

**View names must be alphanumeric** (letters, digits, hyphens, underscores, starting with a letter). Spaces or special characters will be rejected with a console error.

**Pipe separators in multi-view loads must match exactly.** If you have two view names, you need two destination selectors and either zero or two param segments.

**`data-refresh` targets must have `data-view-refreshable`.** If they don't, the refresh event is dispatched but `cba-view` will warn and do nothing.

**`data-load-view` and `data-load-dest` must always be used together.** Using one without the other logs a warning and neither takes effect.

**`cba-api` requires you to add `data-controller="cba-api"` yourself** to a wrapper element. `cba-rest` is already on `<body>` — you don't add it.

**Your view fragments must not include `<script>` tags.** The shell sanitizes all HTML before injecting it, stripping scripts, iframes, inline event handlers, and `javascript:` URLs.

**For polling to work, `data-view-refreshable` must be present along with `data-view-name`.** The polling interval starts only after the element has been loaded via `cba-view#load` or a `cbapp:refresh`/`cbapp:load` event and a sentinel element is found in the response.

**`data-modal-close` on a form closes on success only, not on every submit.** If the operation fails, the modal stays open so the user can fix the problem and try again.

**Do not put `data-modal-close` on a submit button** — it will close the modal on click before the REST call completes. Put it on the `<form>` element for post-action close, or on a `<button type="button">` for immediate cancel.
