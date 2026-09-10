---
title: "Restore"
description: "Restore a previously deleted Content Instance."
---
Restore a previously deleted Content Instance.

## Endpoints

```
PATCH repo/xmsId/{xmsId}/restore
PATCH repo/xmsId/{xmsId}/tag/{tagName}/restore
```

## Path Parameters

| Parameter | Description |
|-----------|-------------|
| `{xmsId}` | The `XmsContentId` of the content to restore |
| `{tagName}` | Optional. Tag of the Content Instance to restore. Defaults to `HEAD` if omitted |

See [Path Parameters](/api/parameters/path) for details.

## Parameters

All [common parameters](/api/parameters/form) apply.

<Info>
[Property Parameters](/api/parameters/property) are ignored for restore operations.
</Info>

## Behavior

- PATCH is used because the operation does not fully specify the resource — it only specifies that the content should be un-deleted.
- When a `pubType` is specified, the operation passes through the associated transform, enabling cascading operations.

## Query Parameters

All [query parameters](/api/parameters/query) apply: `view`, `dryRun`, `deepResults`, `summarize`.

## Response

Returns a `LoadResult` resource (XML or JSON based on `Accept` header), unless the `view` query parameter is set.
