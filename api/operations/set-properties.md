---
title: "Set Properties"
description: "Update properties on an existing Content Instance without providing full content."
---
Update properties on an existing Content Instance without providing full content.

## Endpoints

```
PATCH repo/xmsId/{xmsId}/properties
PATCH repo/xmsId/{xmsId}/tag/{tagName}/properties
```

## Path Parameters

| Parameter | Description |
|-----------|-------------|
| `{xmsId}` | The `XmsContentId` of the destination content to update |
| `{tagName}` | Optional. Tag for the new Content Instance. Defaults to `HEAD` if omitted |

See [Path Parameters](/api/parameters/path) for details.

## Parameters

All [common parameters](/api/parameters/form) apply. Additionally:

- [Property Parameters](/api/parameters/property) (`prop.[name]`) specify the property values to set
- Use `prop.[name].remove` to remove a property value from the Content Instance

## Behavior

- Only the specified properties are changed; unmentioned properties are carried over from the last Content Instance.
- When a `pubType` is specified, the operation passes through the associated transform, enabling cascading operations.

## Query Parameters

All [query parameters](/api/parameters/query) apply: `view`, `dryRun`, `deepResults`, `summarize`.

## Response

Returns a `LoadResult` resource (XML or JSON based on `Accept` header), unless the `view` query parameter is set.
