---
title: "DrawerController"
description: "Slide-in panel (left or right) for the cbapp shell. Mounted on automatically. One drawer per page."
---
Slide-in panel (left or right) for the cbapp shell. Mounted on `<body>` automatically. One drawer per page.

The controller manages open/close state, panel width, the backdrop overlay, keyboard dismissal (Escape), and clearing stale content after close. Each drawer declares its own side; width is chosen per open by the trigger — see [Drawer Width](#drawer-width).

## Drawer HTML Shell

Place this once in your app layout, outside any view-swap container:

```html
<aside class="cbapp-core-ui__drawer" data-drawer-side="right" id="main-drawer">
  <div class="cbapp-core-ui__drawer-header">
    <span class="cbapp-core-ui__drawer-title">Panel Title</span>
    <button class="cbapp-core-ui__drawer-close" data-action="click->cba-drawer#close" aria-label="Close">
      <!-- X icon -->
      <svg ...></svg>
    </button>
  </div>
  <div class="cbapp-core-ui__drawer-content" id="main-drawer-content"></div>
</aside>
```

### Drawer Element Attributes

| Attribute          | Required    | Description                                                                      |
| ------------------ | ----------- | -------------------------------------------------------------------------------- |
| `data-drawer-side` | Yes         | `"left"` or `"right"` — determines slide direction                               |
| `data-drawer-size` | No          | Set by the controller at open time — do not author it by hand. See [Drawer Width](#drawer-width) |
| `id`               | Recommended | Used by `data-drawer-id` on open triggers to target this drawer specifically     |

## Drawer Width

Because there is one drawer per page, width is decided **per open by the trigger**, not baked into the `<aside>`. Add `data-drawer-size` to whatever opens the drawer — a `cba-drawer#open` button or a `cba-view#load` trigger:

```html
<!-- Wide drawer via a view load -->
<a data-action="click->cba-view#load"
   data-view-name="editForm"
   data-view-dest="#main-drawer-content"
   data-drawer-size="lg">Edit</a>

<!-- Wide drawer via a standalone button -->
<button data-action="click->cba-drawer#open" data-drawer-size="lg">Open</button>
```

### Sizes

| Size | Width | Notes |
|---|---|---|
| `sm` | 24rem (384px) | |
| `md` | 32rem (512px) | **Default** — applied when no size is requested |
| `lg` | `min(64rem, 50vw)` | Tracks half the viewport, capped at 1024px |

`sm` and `md` are fixed. `lg` is viewport-relative on purpose: a fixed width stops being "half the screen" on a large display, while a pure fraction becomes an unreadable wall of content on an ultrawide. `min()` gives you half the screen up to 2048px, then holds at a readable 1024px.

| Screen | `lg` renders | Share of viewport |
|---|---|---|
| 1280px | 640px | half |
| 1440px | 720px | half |
| 1920px | 960px | half |
| 2560px | 1024px | 40% |

### The half-viewport ceiling

Every drawer carries `max-width: 50vw`, so **no drawer can exceed half the viewport** regardless of how its width was set — token, app override, or a legacy `w-*` utility class on the `<aside>`. `max-width` always constrains `width`, so this cannot be bypassed by accident.

Below `40rem` (phone widths) the drawer goes full-width instead. Half a 390px screen is unusable, and the ceiling alone can't fix that — the token width is already tiny at that size.

### Overriding sizes in your app

Widths are CSS custom properties, so an app redefines one in a single line with no specificity fight:

```css
/* app CSS, loaded after cbapp-core.css */
aside.cbapp-core-ui__drawer[data-drawer-size="lg"] {
  --cbapp-drawer-width: 60rem;
}
```

The `max-width: 50vw` ceiling still applies. To go wider than half the viewport you must also raise `max-width` — deliberately awkward, because that's the constraint the shell is enforcing.

### Reset behaviour

Size resets to `md` on every fresh open, so a drawer opened wide once does not stay wide. While the drawer is already open, only an **explicit** `data-drawer-size` resizes it — a refresh or poll re-fires the open path with no trigger and must not snap the drawer back to the default.

An unrecognised size logs a console warning and falls back to `md`.

## Opening the Drawer

### Via a view load (most common)

Point `data-view-dest` at the drawer's content div. The drawer opens immediately (showing a spinner) as soon as the fetch starts, then the content replaces the spinner when the response arrives. No extra attributes needed.

```html
<a data-action="click->cba-view#load" data-view-name="editForm" data-view-dest="#main-drawer-content"> Edit </a>
```

### Via a standalone button

Use when you want to open the drawer without loading content through the view controller.

```html
<button data-action="click->cba-drawer#open">Open</button>
```

To target a specific drawer by id (useful if you ever add a second drawer):

```html
<button data-action="click->cba-drawer#open" data-drawer-id="main-drawer">Open</button>
```

## Closing the Drawer

The drawer can be closed in four ways:

| Method         | How                                                                               |
| -------------- | --------------------------------------------------------------------------------- |
| Close button   | `data-action="click->cba-drawer#close"` on any button                             |
| Backdrop click | Automatic — clicking outside the drawer closes it                                 |
| Keyboard       | Pressing `Escape` closes it                                                       |
| REST success   | Add `data-close-drawer` to the `<form>` or a `<button type="submit">` — see below |

Content inside the drawer is cleared **after** the close transition completes, so the previous state is never briefly visible the next time the drawer opens.

### Close on REST success

Add `data-close-drawer` to your form or submit button. The drawer closes automatically after a successful REST operation.

```html
<!-- On the form — closes for any successful submit -->
<form data-rest-content="load" data-close-drawer data-toast-success="Saved!">...</form>

<!-- On the submit button only — cancel button is unaffected -->
<form data-rest-content="load" data-toast-success="Saved!">
  ...
  <button type="submit" data-close-drawer>Save</button>
  <button type="reset">Cancel</button>
</form>
```

## Events

| Event                | Fired when                                                                             |
| -------------------- | -------------------------------------------------------------------------------------- |
| `cbapp:view-loading` | Spinner is injected into the drawer content — this is what triggers the drawer to open |

The controller does not dispatch its own events. Listen for `cba-rest:success` (from RestController) if you need to act after a REST operation that closes the drawer.

## Complete Example

A list page with an "Edit" link that opens a form in a right-side drawer:

```html
<!-- In your layout (once, outside any view container) -->
<aside class="cbapp-core-ui__drawer w-[480px]" data-drawer-side="right" id="edit-drawer">
  <div class="cbapp-core-ui__drawer-header">
    <span class="cbapp-core-ui__drawer-title">Edit Item</span>
    <button class="cbapp-core-ui__drawer-close" data-action="click->cba-drawer#close" aria-label="Close">✕</button>
  </div>
  <div class="cbapp-core-ui__drawer-content" id="edit-drawer-content"></div>
</aside>

<!-- In your view fragment (loaded by ViewController) -->
<a data-action="click->cba-view#load" data-view-name="editItem" data-view-dest="#edit-drawer-content"> Edit </a>

<!-- The form fragment loaded into the drawer -->
<form data-rest-content="load" data-toast-success="Saved!" data-refresh="#items-list">
  <input type="hidden" name="contentId" value="..." />
  <input name="title" />
  <button type="submit" data-close-drawer>Save</button>
  <button type="reset" data-action="click->cba-drawer#close">Cancel</button>
</form>
```
