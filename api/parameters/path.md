---
title: "Path Parameters"
description: "All LCMR REST endpoints share these path parameters."
---
All LCMR REST endpoints share these path parameters.

## Base Path

```
{api-base}/webresources/lcm/repo
```

## Parameters

### `{xmsId}`

The `XmsContentId` of the **destination** content of the write operation — the content that will be operated on.

- Required for all PATCH endpoints
- Not included in POST endpoints (content will be identified later, either by system generation or extraction)

<Tip>
Path parameters identify the *destination* content, not the *source* content. Source content is specified by form parameters (`srcContentId`, `srcFile`).
</Tip>

### `{tagName}`

The name of the tag to apply to the new Content Instance.

- **Optional** — When omitted, `HEAD` is used as the default tag
- When included, the endpoint path contains `/tag/{tagName}` as an additional segment

The ability to create new Content Instances with a `tagName` other than `HEAD` provides incomplete branching capabilities. It remains incomplete because the content history does not preserve what branch tags past Content Instances have had.

## Examples

```
# PATCH to a specific content item (HEAD tag)
PATCH repo/xmsId/abc-123/delete

# PATCH to a specific content item with a named tag
PATCH repo/xmsId/abc-123/tag/review/delete

# POST new content (no xmsId, HEAD tag)
POST repo

# POST new content with a named tag
POST repo/tag/draft
```
