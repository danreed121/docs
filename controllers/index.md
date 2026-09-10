---
title: "Controllers"
description: "The cbapp-core shell provides five Stimulus controllers. All are registered automatically when cbapp-core.js is loaded."
---
The cbapp-core shell provides five Stimulus controllers. All are registered automatically when `cbapp-core.js` is loaded.

| Controller | Registered on | Description |
|---|---|---|
| [ViewController](/controllers/view-controller) | `<body>` | Loads HTML view fragments into destination elements |
| [RestController](/controllers/rest-controller) | `<body>` | ContentBase content operations (load, delete, restore, etc.) |
| [ApiController](/controllers/api-controller) | App root element | HTTP form submissions to client-specific API endpoints |
| [DrawerController](/controllers/drawer-controller) | `<body>` | Slide-in panel (left or right) with overlay and keyboard dismissal |
| [UploadController](/controllers/upload-controller) | Drop zone element | Drag-and-drop file upload with type/size validation |

## Shared Behaviour

All three controllers share the following conventions:

- **Event delegation** — controllers listen on their host element; events bubble from child forms/triggers automatically. No `data-action` needed on forms for rest/api.
- **Confirm gate** — any trigger or form with `data-confirm-msg` shows a confirmation dialog before proceeding.
- **Post-action attributes** — `data-toast-success`, `data-toast-error`, and `data-refresh` work the same way across all controllers.
- **401 handling** — any 401 response automatically redirects to the login screen. No additional handling required.
- **Abort safety** — in-flight requests are aborted if the same trigger fires again or the controller disconnects.
