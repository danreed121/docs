---
title: "File Input Parameters"
description: "File input parameters attach files to an LCMR REST operation. Each file input consists of up to three associated form parameters sharing a common name."
---
File input parameters attach files to an LCMR REST operation. Each file input consists of up to three associated form parameters sharing a common name.

## Parameters

### `filePram.[name]`

The file attachment itself. `[name]` is a text string that acts as the identifier for this File Input and associates the three parameter patterns together.

### `filePramRendition.[name]`

Optional. Selects a rendering from the Publication Type when the file attachment is rendered.

- Defaults to `any / unknown` if omitted

### `filePramContentType.[name]`

Optional. Specifies the content type of the file attachment.

- Defaults to `Unspecified` if omitted

## Behavior

- The `[name]` portion must match across all three parameter patterns to associate them
- If `filePramRendition.[name]` or `filePramContentType.[name]` (or both) are passed **without** a corresponding `filePram.[name]` attachment, a File Input with that name is created with **empty content**

## Examples

```
# Simple file upload
filePram.document = (file data)

# File upload with rendition and content type
filePram.image = (file data)
filePramRendition.image = thumbnail
filePramContentType.image = image/jpeg

# Create an empty file input with just metadata
filePramRendition.placeholder = default
filePramContentType.placeholder = text/xml
```

## Usage with Operations

File inputs are primarily used with the [Load](/api/operations/load) and [Pass](/api/operations/pass) operations. Reference a file input by name using the `srcFile` [form parameter](/api/parameters/form).
