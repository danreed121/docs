---
title: "RestController"
description: "Declarative ContentBase content operations via form submit or button click. Mounted on by the app shell."
---
Declarative ContentBase content operations via form submit or button click. Mounted on `<body>` by the app shell.

Intercepts submit events on `<form data-rest-content="[operation]">` and click events on `<button data-rest-content="[operation]">` (outside a handled form). See the [REST API docs](/api/index) for details on the underlying operations.

## Setup

The controller is registered on `<body>` automatically by the shell. Use a `<form>` when you need field inputs or multipart file uploads. Use a `<button>` for simple operations that only need a few params.

```html
<!-- Button — no form needed for simple ops -->
<button data-rest-content="delete"
        data-content-params='{"contentId":"abc-123"}'
        data-confirm-msg="Delete this item?"
        data-toast-success="Deleted.">
  Delete
</button>
```

## Form Attributes

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-rest-content` | Yes | — | Operation name — see [Operations](#operations) below |
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
| `data-rest-content` | Yes | — | Operation name |
| `data-content-params` | No | `{}` | JSON object of operation fields (e.g. `contentId`, property values) |

## Operations

| Value | Description | Docs |
|---|---|---|
| `load` | Create or update a content node | [Load](/api/operations/load) |
| `setProperties` | Update properties on an existing node | [Set Properties](/api/operations/set-properties) |
| `delete` | Soft-delete a content node | [Delete](/api/operations/delete) |
| `restore` | Restore a deleted content node | [Restore](/api/operations/restore) |
| `pass` | Pass-through without writing | [Pass](/api/operations/pass) |

The `contentId` field value is used as the `xmsId` path parameter. Include it as a hidden input or as a named field in the form.

## Events

All events bubble to `document`. For button triggers, `form` in the event detail is the button element.

| Event | Detail | Description |
|---|---|---|
| `cba-rest:success` | `{ form, operation, result }` | Operation completed successfully |
| `cba-rest:error` | `{ form, operation, error }` | Operation failed |
| `rest:cancelled` | `{ form, operation }` | Confirm dialog was dismissed |

## Examples

### Delete with button (no form needed)

```html
<button data-rest-content="delete"
        data-content-params='{"contentId":"abc-123"}'
        data-confirm-msg="Delete this item?" data-confirm-title="Confirm Delete"
        data-toast-success="Deleted." data-toast-error="Delete failed.">
  Delete
</button>
```

### Delete with form (when you need hidden inputs or other fields)

```html
<form data-rest-content="delete"
      data-confirm-msg="Delete this item?" data-confirm-title="Confirm Delete"
      data-toast-success="Deleted." data-toast-error="Delete failed.">
  <input type="hidden" name="contentId" value="abc-123">
  <button type="submit">Delete</button>
</form>
```

### Load (create/update) with view refresh

```html
<form data-rest-content="load"
      data-toast-success="Saved!"
      data-refresh="#items-list">
  <input type="hidden" name="contentId" value="abc-123">
  <input name="title" value="...">
  <button type="submit">Save</button>
</form>
```

### Listen for success and act on the result

```js
document.addEventListener("cba-rest:success", (e) => {
  const { operation, result } = e.detail;
  if (operation === "load") {
    console.log("Saved content ID:", result.xmsId);
  }
});
```
