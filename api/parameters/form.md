---
title: "Form Parameters"
description: "Standard form parameters available on LCMR REST endpoints. These are in addition to the open-ended Property, Custom Property, Transform, and File Input..."
---
Standard form parameters available on LCMR REST endpoints. These are in addition to the open-ended [Property](/api/parameters/property), [Custom Property](/api/parameters/custom-property), [Transform](/api/parameters/transform), and [File Input](/api/parameters/file-input) parameters.

## Parameters

| Name | Required | Description |
|------|----------|-------------|
| `srcContentId` | No | The `XmsContentId` of the Content Entity to use as input to the transform. Uses the Content Instance specified by `srcTag` (or `HEAD` if `srcTag` is omitted). When no `pubType` is specified (and therefore no transform), this content is used as the direct source for a Load operation; for any other operation it produces an error. Cannot be combined with `srcFile`. |
| `srcTag` | No | Used with `srcContentId` to specify which Content Instance to use for input content. Also used as the context tag for all renderings (the default tag for many extension functions in the entire cascade tree). |
| `viewPath` | Yes (new content), No (otherwise) | For new content, determines the path in the Repository View where the content will appear. May also be passed for other write operations where it could be useful for cascading operations. |
| `pubType` | No | The Publication Type used to find transforms. If omitted, the write operation occurs without a transform and without the opportunity for further cascades. |
| `srcFile` | No | The name of a [File Input](/api/parameters/file-input) to use as input to the transform. When no `pubType` is specified (and therefore no transform), this content is used as the direct source for a Load operation; for any other operation it produces an error. Cannot be combined with `srcContentId`. |
| `restrictionFlags` | No | [Write Restriction Flags](/api/parameters/restriction-flags) that disallow specified kinds of content changes. See the [Restriction Flags](/api/parameters/restriction-flags) reference for the flag format. |

## Mutual Exclusivity

<Warning>
`srcContentId` and `srcFile` are mutually exclusive. Specifying both will produce an error.
</Warning>

## Source Resolution

The source content for an operation is determined by this priority:

1. **`srcContentId`** (+ optional `srcTag`) — Use an existing Content Entity from the system
2. **`srcFile`** — Use a named File Input attachment
3. If neither is specified and a `pubType` is set, the transform runs without explicit source content
4. If neither is specified and no `pubType` is set, the operation proceeds without source content (applicable only to certain operations)
