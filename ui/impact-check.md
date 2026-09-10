---
title: "Impact Check"
description: "A two-stage confirmation gate for destructive operations. Before the actual operation executes, a GET request is sent to the same endpoint to retrieve a..."
---
A two-stage confirmation gate for destructive operations. Before the actual operation executes, a GET request is sent to the same endpoint to retrieve a consequence message from the server (e.g. "This will delete 42 records"). That message is shown in a confirm dialog. If the user cancels, the operation never runs.

Supported by both [RestController](/controllers/rest-controller) and [ApiController](/controllers/api-controller).

---

## How It Works

1. User clicks submit / triggers the action
2. A GET request is sent to the operation endpoint with the same parameters
3. The server response is inspected at `data-impact-message-path` to extract the warning message
4. A confirm dialog shows the message
5. If the user confirms → the real operation executes (POST, DELETE, PATCH, etc.)
6. If the user cancels → operation is aborted, a `cancelled` event is dispatched

---

## Attributes

Place these on the `<form>` or `<button>` alongside the REST or API operation attributes.

| Attribute | Required | Default | Description |
|---|---|---|---|
| `data-impact-confirm` | Yes | — | Presence enables the impact check gate. No value needed. |
| `data-impact-message-path` | No | `"Response.message"` | Dot-notation path into the GET response JSON to extract the warning message shown in the dialog |
| `data-confirm-title` | No | `"Confirm"` | Dialog title |
| `data-confirm-label` | No | `"Continue"` | Confirm button label |
| `data-cancel-label` | No | `"Cancel"` | Cancel button label |

---

## Examples

### REST — delete with impact check

```html
<form data-rest-content="delete"
      data-impact-confirm
      data-toast-success="Deleted.">
  <input type="hidden" name="contentId" value="abc-123">
  <button type="submit">Delete</button>
</form>
```

The GET request goes to the same endpoint as the delete. The server returns something like:

```json
{ "Response": { "message": "This will permanently delete 3 child items." } }
```

That message appears in the confirm dialog. `data-impact-message-path` defaults to `"Response.message"` so no path attribute is needed here.

### REST — custom message path and dialog labels

```html
<form data-rest-content="delete"
      data-impact-confirm
      data-impact-message-path="data.warning"
      data-confirm-title="Confirm Delete"
      data-confirm-label="Yes, delete"
      data-cancel-label="Keep it"
      data-toast-success="Deleted.">
  <input type="hidden" name="contentId" value="abc-123">
  <button type="submit">Delete</button>
</form>
```

Expects a response like `{ "data": { "warning": "This item has 5 references." } }`.

### REST — button trigger (no form)

```html
<button data-rest-content="delete"
        data-content-params='{"contentId":"abc-123"}'
        data-impact-confirm
        data-confirm-title="Confirm Archive"
        data-confirm-label="Archive"
        data-toast-success="Archived.">
  Archive
</button>
```

### API — impact check on a custom endpoint

```html
<form data-api-url="/api/bulk-delete" data-api-method="DELETE"
      data-impact-confirm
      data-impact-message-path="result.consequence"
      data-confirm-label="Delete all"
      data-toast-success="All items deleted.">
  <input type="hidden" name="filter" value="archived">
  <button type="submit">Delete Archived</button>
</form>
```

---

## Message Path

`data-impact-message-path` uses dot-notation to traverse the JSON response. If the path doesn't resolve to a value, the dialog falls back to `"Are you sure you want to proceed?"`.

| Response JSON | Path | Resolved message |
|---|---|---|
| `{ "Response": { "message": "..." } }` | `Response.message` *(default)* | The message string |
| `{ "data": { "warning": "..." } }` | `data.warning` | The warning string |
| `{ "info": { "detail": { "text": "..." } } }` | `info.detail.text` | The text string |
| *(path not found)* | any | `"Are you sure you want to proceed?"` |

---

## Events

| Event | Controller | Detail | Description |
|---|---|---|---|
| `cba-rest:cancelled` | RestController | `{ trigger, operation }` | Impact check dialog was dismissed |
| `cba-api:cancelled` | ApiController | `{ form: trigger }` | Impact check dialog was dismissed |

```js
document.addEventListener("cba-rest:cancelled", (e) => {
  console.log("Cancelled:", e.detail.operation);
});
```

---

## Combining with `data-confirm-msg`

`data-impact-confirm` and `data-confirm-msg` are separate gates and should not be used together on the same trigger. Use `data-confirm-msg` when the warning message is static and known at render time. Use `data-impact-confirm` when the server needs to calculate the consequence (e.g. dependent record counts) before the user commits.

| | `data-confirm-msg` | `data-impact-confirm` |
|---|---|---|
| Message source | Hard-coded in HTML | Fetched from server at click time |
| HTTP round-trip | None | GET before the real operation |
| Use when | Simple "are you sure?" | Server-calculated consequence |
