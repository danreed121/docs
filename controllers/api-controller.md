---
title: "ApiController"
description: "Declarative HTTP requests to client-specific API endpoints via form submit or button click. Not mounted by default — add data-controller=\"cba-api\" to the..."
---
Declarative HTTP requests to client-specific API endpoints via form submit or button click. Not mounted by default — add `data-controller="cba-api"` to the element that contains your forms or buttons.

Use this for app-specific endpoints that are not part of the ContentBase REST API. For ContentBase content operations, use [RestController](/controllers/rest-controller) instead.

## Setup

Add `data-controller="cba-api"` to any ancestor element. Forms and buttons inside with `data-api-url` will be intercepted automatically.

```html
<div data-controller="cba-api">
  <!-- Button — no form needed for simple calls -->
  <button data-api-url="/api/items/123" data-api-method="DELETE"
          data-api-params='{"reason":"outdated"}'
          data-toast-success="Deleted.">
    Delete
  </button>
</div>
```

<Tip>
Scope the controller to the smallest element that contains your triggers rather than placing it on `<body>`. Multiple independent `api` controllers can coexist on the same page.
</Tip>

**Forms inside a drawer:** The drawer `<aside>` sits on `<body>` as a sibling of your app root — outside wherever `cba-api` is mounted. Forms loaded into a drawer will do a native browser POST unless you add `data-controller="cba-api"` directly to the form itself. Stimulus connects it automatically when the drawer content is injected.

```html
<form data-controller="cba-api"
      data-api-url="/api/items"
      data-api-method="POST"
      data-toast-success="Saved!"
      data-close-drawer>
  ...
</form>
```

## Form Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-api-url` | Yes | — | Endpoint path (resolved against `cb-root-url`) or absolute URL |
| `data-api-method` | No | `POST` | HTTP method: `GET` `POST` `PUT` `PATCH` `DELETE` |
| `data-toast-success` | No | — | Toast message shown on success |
| `data-toast-error` | No | `err.message` | Toast message shown on error |
| `data-refresh` | No | — | CSS selector of a view dest element to reload on success |
| `data-confirm-msg` | No | — | Shows a confirm dialog before submit |
| `data-confirm-title` | No | `"Confirm"` | Confirm dialog title |
| `data-confirm-label` | No | `"Confirm"` | Confirm button label |
| `data-cancel-label` | No | `"Cancel"` | Cancel button label |

## Button Attributes

All form attributes above apply to buttons too. Additionally:

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-api-url` | Yes | — | Endpoint path or absolute URL |
| `data-api-method` | No | `POST` | HTTP method |
| `data-api-params` | No | `{}` | JSON object sent as the request body |

## Events

All events bubble to `document`. For button triggers, `form` in the event detail is the button element.

| Event | Detail | Description |
|---|---|---|
| `cba-api:success` | `{ form, result }` | Request completed successfully |
| `cba-api:error` | `{ form, error }` | Request failed |
| `cba-api:cancelled` | `{ form }` | Confirm dialog was dismissed |

## Examples

### DELETE with a button (no form needed)

```html
<div data-controller="cba-api">
  <button data-api-url="/api/items/123" data-api-method="DELETE"
          data-confirm-msg="Delete this item?" data-confirm-title="Confirm Delete"
          data-confirm-label="Yes, delete" data-cancel-label="Never mind"
          data-toast-success="Deleted." data-toast-error="Delete failed.">
    Delete
  </button>
</div>
```

### POST with params and view refresh (button)

```html
<div data-controller="cba-api">
  <button data-api-url="/api/items/123/publish"
          data-api-params='{"notify":true}'
          data-toast-success="Published!" data-refresh="#items-list">
    Publish
  </button>
</div>
```

### POST with form fields (use a form for user input or multipart uploads)

```html
<div data-controller="cba-api">
  <form data-api-url="/api/items" data-api-method="POST"
        data-toast-success="Saved!" data-refresh="#items-list">
    <input name="title" value="...">
    <button type="submit">Save</button>
  </form>
</div>
```

### PATCH with view refresh (form)

```html
<div data-controller="cba-api">
  <form data-api-url="/api/items/123" data-api-method="PATCH"
        data-toast-success="Saved!" data-refresh="#items-list">
    <input name="title" value="...">
    <button type="submit">Save</button>
  </form>
</div>
```

### Listen for success and act on the result

```js
document.addEventListener("cba-api:success", (e) => {
  console.log("Response:", e.detail.result);
});
```

## Programmatic Usage

For cases where you need before/after logic, use `ContentBase.callApi` directly inside your own Stimulus controller:

```js
import { Controller } from "@hotwired/stimulus";

export default class extends Controller {
  async save() {
    // do something before...
    const result = await ContentBase.callApi("/api/items", {
      method: "POST",
      body: { title: "Hello" },
    });
    // do something after...
  }
}
```

`callApi` handles URL resolution, standard headers, JSON/multipart body serialization, and 401 redirect automatically.
