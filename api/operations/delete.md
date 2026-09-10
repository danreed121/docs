---
title: "Delete"
description: "Soft-delete a Content Instance. The content is marked as deleted but can be restored."
---
Soft-delete a Content Instance. The content is marked as deleted but can be restored.

## Endpoints

```
PATCH repo/xmsId/{xmsId}/delete
PATCH repo/xmsId/{xmsId}/tag/{tagName}/delete
```

## Path Parameters

| Parameter | Description |
|-----------|-------------|
| `{xmsId}` | The `XmsContentId` of the content to delete |
| `{tagName}` | Optional. Tag of the Content Instance to delete. Defaults to `HEAD` if omitted |

See [Path Parameters](/api/parameters/path) for details.

## Parameters

All [common parameters](/api/parameters/form) apply.

<Info>
[Property Parameters](/api/parameters/property) are ignored for delete operations.
</Info>

## Behavior

- PATCH is used instead of HTTP DELETE for consistency with other operations.
- When a `pubType` is specified, the operation passes through the associated transform, enabling cascading operations.
- Nested entities or owned relationships may trigger additional deletions when documents are involved.

## Query Parameters

All [query parameters](/api/parameters/query) apply: `view`, `dryRun`, `deepResults`, `summarize`.

## Response

Returns a `LoadResult` resource (XML or JSON based on `Accept` header), unless the `view` query parameter is set.
