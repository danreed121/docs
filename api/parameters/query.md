---
title: "Query Parameters"
description: "Optional query parameters available on all LCMR REST endpoints."
---
Optional query parameters available on all LCMR REST endpoints.

## `view`

When present (regardless of value), the result of the transform is returned as the HTTP response instead of a `LoadResult` resource.

- Requires `pubType` to also be specified — if no transform exists, there is nothing to use as a response, and an error is returned
- When omitted, the response is a `LoadResult` resource in XML or JSON (as dictated by `Accept` headers)

```
PATCH repo/xmsId/abc-123/load?view
```

## `dryRun`

When set to `true`, the write operation executes fully but throws an exception at the very end, causing all changes to be **rolled back**. The response (whether a view or `LoadResult`) is returned as if the operation had been executed.

- Response code: **202 Accepted**
- Useful for previewing the result of an operation without committing changes

```
PATCH repo/xmsId/abc-123/load?dryRun=true
```

## `deepResults`

When set to `true`, only the first `ContentResult` (root, if applicable) is included in the `LoadResult`.

This parameter originated from scenarios like document loads where nested entities or relationships to other owned entities might trigger deletions when omitted during updates. In these cases, the first result is the root entity of the document. In newer API use cases, there may not be a clear root result, so the first result can be somewhat arbitrary.

The `LoadResult` does not distinguish between levels of nesting or indirect operations (like delete due to omission).

```
PATCH repo/xmsId/abc-123/load?deepResults=true
```

## `summarize`

When present, the `LoadResult` does not include any `ContentResult` structures. Instead, it returns only counts of the different operation types:

- Add
- Update
- Delete
- Restore

```
PATCH repo/xmsId/abc-123/load?summarize
```
