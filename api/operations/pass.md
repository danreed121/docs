---
title: "Pass"
description: "Pass content through a transform without performing a direct write operation. Used when the transform (XSLT) contains the logic to determine and operate on..."
---
Pass content through a transform without performing a direct write operation. Used when the transform (XSLT) contains the logic to determine and operate on content.

## Endpoints

### PATCH — Identified Content

```
PATCH repo/xmsId/{xmsId}/pass
PATCH repo/xmsId/{xmsId}/tag/{tagName}/pass
```

### POST — Unidentified Content

Used when the XSLT entirely contains the logic to determine the content to be operated on.

```
POST repo/pass
POST repo/tag/{tagName}/pass
```

## Path Parameters

| Parameter | Description |
|-----------|-------------|
| `{xmsId}` | The `XmsContentId` of the destination content |
| `{tagName}` | Optional. Tag for the Content Instance. Defaults to `HEAD` if omitted |

See [Path Parameters](/api/parameters/path) for details.

## Parameters

All [common parameters](/api/parameters/form) apply. Additionally:

- [Transform Parameters](/api/parameters/transform) are passed to the XSLT transform
- [File Input Parameters](/api/parameters/file-input) can provide input content

## Behavior

- **PATCH** requires an `xmsId` identifying the destination content.
- **POST** is used when content identification is handled entirely by the transform logic.
- A `pubType` is typically specified to select the transform to execute.

## Query Parameters

All [query parameters](/api/parameters/query) apply: `view`, `dryRun`, `deepResults`, `summarize`.

## Response

Returns a `LoadResult` resource (XML or JSON based on `Accept` header), unless the `view` query parameter is set.
